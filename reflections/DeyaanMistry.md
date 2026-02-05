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


