## My Major Contributions

**Most significant commits:**

[1b6d18ba77fd898d7f0aa47f142a76e0336eab98](https://github.com/lse-ds105/group-project-ds105a-2025-ctraltelite/commit/1b6d18ba77fd898d7f0aa47f142a76e0336eab98) Interactive map created. 
[8e385a0ce5e29628830c80bd3704e447219a36f0](https://github.com/lse-ds105/group-project-ds105a-2025-ctraltelite/commit/8e385a0ce5e29628830c80bd3704e447219a36f0) Three base maps of each area created. 


**What I implemented**

I designed all the visualisations we intend to use for our website and the scoring system we will use to measure the safety of each route. This includes; a basic bar plot, comparing the relative safety of the first leg of a route from each club; three maps of each area, which shows the routes from clubs to destination and an overlayed grid showing the level of safety along the route; and also a fully interactive map showing the location of each club, each route and all the areas where crimes were reported. 

1. The scoring system
    - Connect to the database so I can pull all route level data
    - Created crime 'rates', to allow for a more even comparison along each route
    - Assign weights, I believe that exposure to crime so crime rates should be a higher weight than the inconvience of a longer journey, thus the according 0.3 and 0.2 weghts were applied
    - I then created a simple 0-100 safety scale that was easy to interpret 
    - However, when I referenced back to my understanding the database, I saw that there were three different reported crimes
    - With this I decided to apply a crime weighting, such that more serious crimes have a higher score thus if it occurs on a route the route is less safe than one with a less serious crime. After 
2. 
