## My Major Contributions

**Most significant commits:**

[6a4af79b586470950c8589096f8efb6a135e4060](https://github.com/lse-ds105/group-project-ds105a-2025-ctraltelite/commit/6a4af79b586470950c8589096f8efb6a135e4060) Fully automated data collection approach finalised.

[e357c185dcfce456e9b7a062550258c31735c20f](https://github.com/lse-ds105/group-project-ds105a-2025-ctraltelite/commit/e357c185dcfce456e9b7a062550258c31735c20f) Provisional Database Schema completed.


**What I implemented**

I designed and implemented a reproducible, cost-limiting, multi-stage data collection and preprocessing pipeline that integrates Google Places API, Google Routes API, and UK Police API responses. Additionally, I designed a preliminary database schema mapping table relationships. 

1. Gather Nightclub Data
    - Requests Google Places API using textQuery for 3 areas in London. 
    - Validates the API response to ensure that returned locations are nightclubs/bars and that their postcodes are accurate.
    - Creates a subdir called `area_name` to store the nightclub data.

2. Gather Stations Data
    - Requests Google Places API per nightclub within each area. 
    - API fieldmask limits response to 20 stations within a 1 kilometre radius from a given nightclub. 
    - Creates a subdir called `stations` within each `area_name` directory to store the stations data.

3. Filter Stations Data 
    - Divides the area surrounding the club into eight 45 degree sections. 
    - For each section, takes the two closest stations to the nightclub **as well as** always including train, subway, and transit stations.
    - Creates a subdir within `stations` called `filtered_stations` to store the filtered stations data.

4. Gather Routes Data
    - Requests Google Routes API per nightclub-station pairing to generate routes data.
    - Creates a subdir within `area_name` called `routes` to store the routes data.

5. Prepare Routes Data for Police API
    - Reduces the number of route vertices present in the Google Routes API response polyline, preserving every third coordinate only. 
    - Converts the polyline into a polygon by extending the route polyline coordinates perpendicularly by 20m each side (total corridor width of 40m).
    - Converts the polyline coordinates into an acceptable format for Police API. 
    - Creates a separate metadata json preserving metadata for each route. File names for police_params and metadata correspond to one another e.g. `amazing_grace_london_bridge_route_0_metadata.json` and `amazing_grace_london_bridge_route_0_police_api_param.txt`.
    - Creates a `police_params` and `metadata` subdir within each `area_name` to store metadata and police params.

6. Gather crime data
    - Requests UK Police API for crime data within each route corridor for six months, from 2025-01 to 2025-06. 
    - Creates a `crime_data` subdir within each `area_name` which has a subdir for each month's crime data e.g. `2025-01`

**Technical Decisions**

_Calculating Distance_

Straight-line distances between nightclubs and stations were computed using the haversine formula. This straight-line distance allowed me to select only the two closest stations within a given sector (see step 3), therefore avoiding unnecessary Google Routes API calls in later steps.

_Ensuring Good API Citizenship_

Although the Police API is rate-unrestricted and free, I aimed to minimize unnecessary complexity. The downsampling of the google routes polyline (step 6) was necessary because the corridor polygon that I later computed (step 6) creates more vertices (two opposite coordinates perpendicular to the polyline coordinates). As such, a polyline with 13 coordinates becomes one with 27; this increase was particularly significant when automating data collection.

_Minimizing Costs_

Given that Google Maps APIs are billable, I took every measure to limit billable events, including:
    - Specifying a limited FieldMask to return only the data I needed.
    - Only gathering a maximum of 2 bus stations per directional sector so as to limit the number of Google Routes API calls whilst maintaining a geographic spread of routes.
    - Cached data.

_Preserving Route metadata_

When creating the Police API params from a route polygon, I wanted to store them as a `txt file` to preserve differentiation between different file types. In line with this practice, I thought it cleaner to remove all metadata from the police_param, instead choosing to store it in a separate folder with a corresponding filename for ease of use in NB02.

_Modelling an appropriate corridor_

Unable to find accurate data on width of roads/streets in London, I had to decide on an appropriate corridor width for the route polygon to call the Police API. I ultimately settled on 40m, balancing the need for a corridor wide enough to capture significant crime data without distorting results.

_Creating a scalable approach_

In order to make this pipeline as scalable as possible, I aimed to avoid silent failures / likely API crashes. As such, throughout the pipeline I use warnings to ensure that any failures in data collection were easily visible.

_Creating a clear, usable file system_

In order to avoid an unclear directory containing hundreds of different files, I designed a clear directory system so that data can be individually inspected or even regathered without rerunning the entire pipeline.


**Challenges and Solutions** 

_Corridor creation_

I wanted the polygon that I computed from the Google Routes API response (to use for the Police API) to reflect the full route rather than simply capturing an unnecessarily large area containing the coordinates within the polyline. This would massively overestimate the exposure to crime along a given route. 

To do so, I created a corridor around the polyline by extending each route segment (represented by a pair of latitude-longitude coordinates) by a given distance. Within the function `create_corridor`, I derive a perpendicular vector to the direction of two consecutive points in the polyline, before extending the points in a perpendicular direction by a given distance. One degree of latitude was assumed to correspond to approximately 111,320 metres, while the longitudinal conversion was adjusted by the cosine of the latitude to account for meridian convergence. Given the short spatial extent of the routes (typically under 1.5 km), this approximation introduces negligible distortion. 

I ignored duplicate or segments of no length in the polyline, so overall this function improves the accuracy of our data whilst integrating with the Police API.

_Creating an efficient pipeline - integrating API responses_

I wanted to create an efficient a data collection pipeline as possible, minimizing avoidable data transformation procedures for my team. I noticed that the Google Routes API normally returns an encoded polyline but can also return a GeoJsonLinestring. I selected the latter option because it returns coordinates that I could plug directly into my Police API call rather than having an extra step of converting an encoded polyline. Subsequently, I requested the Police API using a polygon that matched the entire route of the Google Routes API rather than individual streets / areas.     

_Initial Database Schema Design_

In previous practice exercises / assessments, I had not had to deal with many-to-many relationships. As such, the relationship between crimes and routes was difficult for me to incorporate into my database schema (e.g. one crime could fall within many routes, one route could incorporate many crimes). I decided on a junction table, `route_crimes`, where each row represents one occurrence of a crime falling within a unique route corridor, to act as an intermediary between `routes` and `crimes`.

## Learning Integration

**Mini-Project-1 Positive Feedback:**

`API limitation awareness: You clearly documented the OpenWeather API limitation (data only available from 2020-11-25) and explained how this informed your decision to reframe your analysis to 2021-2025. Your reflection shows good understanding of data science workflow considerations.`

**Action Taken:**

Although I did not encounter significant API limitations for this project, I endeavoured to maintain high-standards of API research prior to data collection. This resulted in: 
- Employing a fieldmask to reduce costs of Google Maps APIs. 
- Discovering the possibility to have Google Routes API return a GeoJsonLineString.

**Mini-Project-1 Area for Improvement:**

`Data validation: You did basic data inspection (checking data length), but you didn't use more comprehensive validation methods like df.head(), df.info(), df.describe(), or df.shape to understand your data structure better.`

**Action Taken:**

Although my use of fieldmasks ensured a fairly easily-comprehensible data structure, data validation checks formed an important part of my data collection approach. For example, in step 1 I validate returned nightclubs against official postcodes for that area and primaryType. This arose from a realisation that Google Maps Places API can return data outside of the specified location when closely inspecting test API responses. 