# Geospatial analysis on Milan's rent prices
This project was made for the course of Geospatial analysis and representation for data science in Universit√† di Trento, 2022. \
The aim of this project is to perform an analysis of the rents in the city of Milan, divided in neighborhoods. In particular, this project aims to confirm whether spatial dependance is present in the territory and how an area can influence its neighboring ones (if positively or negatively). \
Moreover, after having found the most influential neighborhoods, a study will be performed on the presence of services in some selected areas.
In particular, the average distance from a random point to a metro station in the area will be computed, in order to check if, given the cost of rent, an area is more or less connected.

In detail, the project is developed in the following steps, which is the ideal order to view this project:

1. **Data preparation**: this part contains code which shows the methods followed to retrieve data and/or prepare it for further analysis, in order to work as smoothly as possible.
2. **Data visualization**: some data visualization is performed in order to view the distribution of the services in the territory of the city of Milan.
3. **Descriptive spatial statistics**: an analysis is performed to check if spatial dependence is present and how each area influences its neighbors.
4. **Closest metro**: an analysis is performed to find the average waling distance of a metro stop from any point in three selected neighborhoods (Centro, Ticinese and Famagosta).

## Packages used
**Python**: beautifulsoup, request, pandas, geopandas, pyrosm, osmnx, numpy, shapely, matplotlib, seaborn \
**R**: sf, tmap, spdep, spatialreg
