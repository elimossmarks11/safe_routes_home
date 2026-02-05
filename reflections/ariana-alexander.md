## My Major Contributions

**Most Significant Commits:**
[Fb74d2d81185b314a17375c5b0ab8db6b065762a] (https://github.com/lse-ds105/group-project-ds105a-2025-ctraltelite/commit/fb74d2d81185b314a17375c5b0ab8db6b065762a) 'Completed data transformation with database'.

[a3608c8f35145187957f1a3a00355eb453321293] (https://github.com/lse-ds105/group-project-ds105a-2025-ctraltelite/commit/a3608c8f35145187957f1a3a00355eb453321293) 'data collection of night services complete'. 


**What I Implemented:**
I designed and implemented a night service viability filter and a relational database transformation pipeline that filters routes based on transport accessibility and structures all collected data into a queryable SQL database with foreign key relationships and batch insertion optimisation. 

1. Check Night Service for Stations (NB01 Section 7)
- Queried TfL API to determine night service availability at each station using a 150-metre radius search to find TfL stop point IDs near station coordinates. 
- Checked for three types of night service: Night Tube lines (Central, Victoria, Jubilee, Northern, Piccadilly), Night Bus routes (N-prefixed routes), and 24-hour bus routes (61 routes including 24, 188, 242, 277, 344, etc.).
- Validated routes by examining the `lines` field in TfL's `/StopPoint` API response rather than relying on the unreliable `serviceTypes=Night` parameter. 
- Created a `night_service` subdir within each `area_name` directory containing JSON files with viability status for each route (e.g., farrs_dalston_route_0_night_service.json).
- Successfully identified 159 viable routes (82.0%) with night service and flags 35 non-viable routes (18.0%) for exclusion from the database.

2. Create SQL Database (NB02)
- Filtered collected data to include only routes with night service availability by loading night service JSON files created in NB01 Section 7. 
- Created a relational database schema with six tables (`areas`, `clubs`, `destinations`, `routes`, `crimes`, `route_crimes`) following the provisional schema design with proper foreign key constraints to maintain referential integrity.
- Implemented hierarchical insertion respecting foreign key dependencies: areas → clubs → destinations → routes → crimes, ensuring parent records exist before inserting dependent records
- Used batch insertion operations with `.executemany()` within each table to improve efficiency, collecting all records for a table (e.g., all destinations for one club) before inserting in a single database operation.
- Retrieved auto-generated primary key IDs by querying the database after each batch insertion (e.g., `SELECT destination_id FROM destinations WHERE club_id = ? ORDER BY destination_id DESC LIMIT ?`) rather than calculating them arithmetically, ensuring correct ID mapping for foreign key relationships.
- Linked crimes to routes through a junction table (`route_crimes`) to handle the many-to-many relationship where one route can have multiple crimes and one crime location may fall within multiple overlapping route corridors.
- Loaded 159 viable routes, 4,055 crimes, and their linkages into `data/processed/nightclub_safety.db`. 
- Implemented database verification queries to check table row counts, sample data display, and JOIN operations across related tables. 


**Technical Decisions:**
1. Collecting data on night service at bus stops/tube stations (our destinations)
I added Section 7 to NB01 to collect data on night service availability for each route, as routes to stations/bus stops without Night Tube or Night Bus service would be pointless for nightclub-goers returning home in the early hours. I queried the TfL API to check whether stations have night service.
The initial approach used TfL's `serviceTypes=Night` parameter to filter routes, but this proved unreliable as it returned zero results even for stops that definitely do have night service (e.g. Dalston Junction which serves the N242, N277 and N38). After debugging, I discovered that TfL's API response often had missing `modeName` fields, and more importantly that the `serviceTypes` filter was not capturing 24-hour bus routes. I, therefore, decided to match `/StopPoint` line names against known night services: Night Tube lines (Central, Victoria, Jubilee, Northern, Piccadilly), any route starting with 'N' (night buses) and a list of 24-hour bus routes from. I got this information: night tubes - https://www.london.gov.uk/transport/rail-and-underground/night-tube; 24 hour buses - https://bus-routes-in-london.fandom.com/wiki/List_of_24-Hour_Routes.

2. Not caching TfL night service data
While Eli has used caching in his API calls above, I have not used caching here because TfL API does not require an API key (unlike Google Places API), has simple endpoints and data changes more, with night service routes being added/removed, compared to nightclub locations which are stable.

3. checking night service at stations with stop points vs. lat/lon
Google API produces station coordinates (lat/lon) but TfL needs a stop point ID to query night service. I introduced a radius of 150 metres as a station may be recorded as 51.5050 in our data and 51.5052 in TfL. This way we avoid missing data because of variations in the APIs. Additionally, a single station, such as London Bridge, may have multiple stop points (tube platforms, bus stops, etc.). The radius will capture these additional stop points.

4. Creating a database to store our data: 
Using Week 08 lecture and sqlite3 python documentation (https://docs.python.org/3/library/sqlite3.html), I used typical SQL functions to create this database. `conn.commit()` is an important function I learnt that allows me to make changes, insert rows and rollback and only when I `commit()` are changes permanent. To follow SQL documentation convention, I have capitalised the keywords such as CREATE TABLE or TEXT NOT NULL. This is conventional as it makes it easier to read complex queries.

5. Batch insertions into the database
I debated whether I should use for loops or vectorisation here, given in Week 05 we were taught to use vectorisation wherever possible for efficiency purposes and good practice. I discussed with claude whether this was possible given I do have foreign key dependencies which make vectorisation difficult. Claude suggested a good idea: insert tables separately in dependency order so as to maintain referential integrity, while at the same time, implement batch insertions using `.executemany()` within each table, rather than row-by-row loops (https://docs.python.org/3/library/sqlite3.html#sqlite3.Cursor.executemany). This approach works by first collecting all records for a given table (e.g., all destinations for one club), then inserting them in a single database operation, and finally using SQLite's `cursor.lastrowid` to calculate the range of auto-generated IDs for linking to dependent tables. For example, if inserting 10 destinations returns lastrowid = 25, the IDs are 16-25, allowing me to batch insert routes with destination_id values 16-25 respectively (https://docs.python.org/3/library/sqlite3.html#sqlite3.Cursor.lastrowid). This allows us to achieve the efficiency benefits of batch processing, significantly reducing database overhead when inserting thousands of crime records and hundreds of routes compared to individual `.execute()` calls in loops.


**Challenges and Solutions:**
1. self-containing TfL API call
Initially I faced an error in that `area_slugify` was not defined. Although Eli had defined it earlier in his code, I wanted to avoid running his code as this would change the data he had already collected and would require me getting my own Google API key. I learnt this from a previous mistake I made in which I ran all of NB01 again once he had handed it over to me and the re-running had failed. Thus to solve this error I made Section 7 completely self-contained. Given the `area_slugify` function is so simple (just converts names to lowercase with underscores) it was fine to define it twice, and the function does the same thing in the Section 1 and Section 7 so there is no conflict.

2. check_station_has_night_service
With my initial approach, only 13 out of 194 routes were found to be viable and I faced the error "429 Client Error" (Rate limiting). I increased the delays to 0.5 seconds between checking different stops at the same station and 2 seconds between checking different routes. If it hits a 429 error it will automatically retry with a 5 second wait. The final implementation successfully identified 159 viable routes (82.0%) and 35 non-viable routes (18%); the non-viable routes are filtered out in NB02 (Data Transformation).

3. Negative crime IDs 
Upon checking through my database, I found that the route_crimes junction table contained negative crime IDs (such as -1, -37, -38), which were invalid for primary keys. This occured because I initially attempted to calculate the range of auto-generated crime IDs after batch insertion using `first_crime_id = cursor.lastrowid - len(crimes_to_insert) + 1`. However, after reading the Python sql 3 documentation it specifies that `cursor.lastrowid`'s behaviour with `.executemany()` is unreliable and can return None or unexpected values, and this is what resulted in negative starting IDs, which then propagated to the subsequent crime records for that route. To resolve this, I changed the approach to quering the database after each batch insertion, retrieving the most recently inserted crime records, and reversing this list to give the IDs in insertion order.


## Learning Integration

**Postive Feedback from mini-project 1 and 2:**
**Excellent vectorised operations:** You used vectorised pandas operations consistently throughout your transformations, including `pd.json_normalize()` for efficient JSON flattening and `.groupby()` with `.agg()` for quarterly aggregation. Your reflection shows clear understanding: "loops process data one item at a time in Python, which is much slower and for 40,993 hourly records, this vectorised operation is much more efficient."

**Action Taken:**
In this project, I applied vectorisation principles where appropriate while recognising their limitations for certain operations. While I used batch operations (.executemany()) for efficient database insertions of independent records like crimes, I appropriately chose for loops for operations requiring complex conditionals and file I/O, such as filtering route. 

**Area for Improvement from mini-project 1 and 2:**
**Database Storage:** Some students stored their collected data in a SQL database, which demonstrated advanced data management skills and made their analysis more scalable.

**Action Taken:** 
I implemented a SQL relational database to store all collected data, addressing the feedback to demonstrate advanced data management skills. The database uses a normalized schema with six tables linked by foreign keys, enabling efficient querying and analysis while maintaining referential integrity. I used batch insertion operations with `.executemany()` to optimise performance when loading the routes and crimes, and implemented comprehensive verification queries to ensure data completeness and correct foreign key relationships. This structured approach makes the analysis more scalable and allows for complex SQL queries across related entities (e.g., finding all crimes along routes from a specific nightclub or comparing crime density between areas).