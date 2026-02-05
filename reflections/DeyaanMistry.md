## My Major Contributions

**Most significant commits:**

[1b6d18ba77fd898d7f0aa47f142a76e0336eab98](https://github.com/lse-ds105/group-project-ds105a-2025-ctraltelite/commit/1b6d18ba77fd898d7f0aa47f142a76e0336eab98) Interactive map created. 
[8e385a0ce5e29628830c80bd3704e447219a36f0](https://github.com/lse-ds105/group-project-ds105a-2025-ctraltelite/commit/8e385a0ce5e29628830c80bd3704e447219a36f0) Three base maps of each area created. 


**What I implemented**

I designed all the visualisations we intend to use for our website and the scoring system we will use to measure the safety of each route. This includes; a basic bar plot, comparing the relative safety of the first leg of a route from each club; three maps of each area, which shows the routes from clubs to destination and an overlayed grid showing the level of safety along the route; and also a fully interactive map showing the location of each club, each route and all the areas where crimes were reported. 

1. The scoring system
    - Connect to the database so I can pull all route level data.
    - Created crime 'rates', to allow for a more even comparison along each route.
    - Assign weights, I believe that exposure to crime so crime rates should be a higher weight than the inconvience of a longer journey, thus the according 0.3 and 0.2 weghts were applied.
    - I then created a simple 0-100 safety scale that was easy to interpret 
    - However, when I referenced back to my understanding the database, I saw that there were three different reported crimes.
    - With this I decided to apply a crime weighting, such that more serious crimes have a higher score thus if it occurs on a route the route is less safe than one with a less serious crime. After a quick chat with a friend who is a law student I was able to rank the crimes and assign a relevenat score.
2. Testing the usability of the scoring system 
    - Create a couple tables to help me understand if the scoring system works. 
    - Ensure routes with lower crime exposure and reasonable distance/time are ranked as safer.
    - Confirms the rankings align with intuitive expectations of route safety.
3. Data Visualisations
    - My first main idea was a simple Facet Grid plot, as I had become quite accustomed to using them from my mini project 2 and lecture W05. 
    - This compares average route safety across clubs and areas on a single scale using a colour gradient to quickly identify which clubs offer safer routes home overall rather than only relative to their own area.
    - Next I created an overall block of code that would enable me to create a map which maps the route from club to destination. Within this is also the creation of a square grid with each square that a route passes through having an aggregated safety score attributed to it. This allows for a visual representation of how safe that route and small area is. 
    - My feedback in mini-project 2 told me that I repeated code, so instead of copying the code out three times for each area to produce a map, I instead created this overall block and from there, I could create individual cells for each area with two lines of code allowing me to create unique titles for each graph.
    - The last visual representation was the creation of an interactive map 
    - This plots all three other maps onto a world map, obviously we are only focussing on 3 areas in London. 
    - It shows all the routes, with information on the starting club, destination, number of crimes along the route, distance and duration.
    - Another interactive part is the ability to hover over a club and see the average route safety score, effectively how safe it is to travel home from that club.
    - It also shows crime squares, effectively where crimes along routes were actually recorded, with the ability to see the number of each different crime committed in a 100m^2 area.

    **Techinal Decisions** 

    _Extracting from the Database_

    To generate the dataset used for scoring, I wrote a SQL query that joins areas, clubs, destinations, and routes, and then left-joins the route–crime junction table. I used a LEFT JOIN so that routes with no associated crimes were retained in the data. I aggregated crimes at the route level using COUNT(rc.crime_id) and grouped by route identifiers and metadata, producing one observation per route with all relevant information needed for scoring.

    _Crime Rates_

    I decided to create route-level features that captured both exposure to crime and practical route characteristics. Rather than relying on raw crime counts, I calculated crimes per kilometre and crimes per minute, which allowed routes of different lengths and durations to be compared realtive to others. I also retained distance and duration as separate features so that excessively long or slow routes were penalised even if crime exposure was low. 

    _Route Safety Score_

    I combined the normalised features into a single risk score using a weighting system that prioritises crime exposure over distance and duration. I then inverted this risk score to create a safety score and rescaled it to a 0–100 range to improve interpretability. Routes were ranked within each area by sorting on safety score.

    _Overlaying the Routes_ 

    To create the route overlay, I used GeoPandas and Shapely to convert stored route polylines into geometric LineStrings and projected all spatial data to Web Mercator (EPSG:3857) to support distance-based operations and basemap overlays. I created regular spatial grids and sampled points along each route, assigning safety scores to those points. By spatially joining sampled points to grid cells and averaging scores, I produced a continuous spatial representation of route safety that handled overlapping routes.

**Challenges and Solutions** 

_Data type inconsistencies and cleaning_

One variable extracted from the database was stored as a string, the duration (e.g. "16s") rather than numeric values. This caused errors in calculations such as crime rates per minute. The issue was addressed by explicitly cleaning and standardising data types in Python, stripping non-numeric characters and coercing values to numeric formats, while handling missing and invalid values robustly.

_Feature comparability and normalisation_

Another challenge was ensuring that route-level metrics were comparable both within and across areas. Raw crime counts and distances varied substantially by area, which risked biasing the scoring system. To address this, I applied min–max normalisation within each area, allowing routes to be ranked relative to realistic local alternatives rather than absolute city-wide values. This design choice ensured fair comparisons and more interpretable rankings.

_Spatial geometry construction and projection_

The route data was initially stored as coordinate lists rather than spatial geometries, preventing direct spatial analysis. I resolved this by converting stored polylines into Shapely LineString objects and using GeoPandas to manage spatial data. Additionally, all geometries were reprojected from latitude–longitude coordinates (EPSG:4326) into Web Mercator (EPSG:3857), enabling accurate distance-based operations and alignment with basemap tiles.

_Mapping environment and visualisation limitations_

Finally, integrating basemaps and interactive visualisations exposed limitations related to network access and execution environments, particularly when external tile servers were unavailable. To mitigate this, I adopted a flexible approach that combined static GeoPandas maps with interactive Folium maps and adjusted tile usage where necessary. This ensured that spatial outputs remained viewable and interpretable across different environments.

## Learning Integration 

**Mini-project 2 Feedback**

'Code Duplication: Within your NB01, you defined the function journey_data() 12 times instead of defining it once and reusing it throughout. Once you have created a function, there is no need to define it again. The def journey_data(....): function can be run once and then reused throughout your notebook.' 

To avoid making the same mistake again with this project and duplicate code, within my visualistaions my original plan to create all three plots for overlapped maps was to duplicate the code three times and just chnage the area name and title. However, this time I created an overall framework, that could be applied to all of the areas. Thus removing the need for duplicate feedback. 

**Mini-project 2 Feedback** 

'Note from Jon: In addition to the points made by the first marker, I would point out that the use of colour in your first plot was not the best. The 'bad' result is an increasingly lighter colour, which strikes me as counter-intuitive. You could have used a colour palette with a single colour, going from say, dark blue for the worst result to light blue for the best result.'

This time when I made my plots, as this my main job thus most other areas for improvement aren't relevant re data collection or transformation, I tried to make my colour scaling easier to interpret. SO this time when creating my safety score from 0-100, I used a colour gradient from red to green, so there is a large variation of colour and red most specifically is seen as bad therefore 0. 