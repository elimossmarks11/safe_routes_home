## Curiosity-Driven Exploratory Question
'What are the safest routes for women walking home at night in London?'

<img src="women_safety_perception_stat.png" alt="description" width="600">


## Justification of Data Source Choice

**UK Police API** ➡️ provides the area/road segement for street crimes (filter for crimes under 'violence and sexual offences' category, which women are disproportionately victim to). Query with force-level parameters (filtering for crimes reported by only the City of London Police and the Metropolitan Police) for data on just London. 

**Google API** ➡️ use the Google Area Insights API identify popular night out locations in London (from which may women start their journeys home at night) as well as the Google Places API to collect data on shops number and location of shops open after 23:00. 


## Technical Implementation Plan

**1 - Generate routes** 

Use Google Maps to create realistic walking routes from popular Soho night-out locations to nearby transport hubs, most likey tube stations.


**2 - Collect safety data**  

Combine police-recorded crime data with information on nearby shops and venues that are open late at night, which we have decided to be past 11pm. 

**3 - Link data to routes**  
 
Match crimes and open venues to different parts of each route to measure how safe each section appears.


**4 - Score and display routes**  
 
Create an overall safety score for each route and show safer and less safe routes using maps and rankings.


## Work Distribution Strategy

<img src="Project Coordination Chart.png" alt="description" width="600">

## Risk Mitigation Strategies

| Risk | Mitigation Strategy |
|:--------:|:-------------------:|
| Google API has costs above 10,000 billable events | Store non time-dependent data locally (e.g. locations) |
| Cases of violence and sexual offences are notoriously underreported | This trend is not area-dependent and will not skew comparison between areas. |
| Police API data is declared monthly | Our findings will be presented as a risk indicator, not a guarantee. |
| Safest routes might be unreasonably long | Impose a duration cap, present these routes as 'balanced' |

<img src="incident_reporting_stats.jpeg" alt="description" width="600">
