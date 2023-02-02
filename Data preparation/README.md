# Description of the files
Packages used: pandas, BeautifulSoup, request, geopandas, matplotlib
- getData: the data was scraped using BeautifulSoup from [this resource](https://www.immobiliare.it/mercato-immobiliare/lombardia/milano/), which is an Italian website for listing houses on sale and to rent. The resource is a table listing the average prices of rent and sale per square meter, divided in the different neighborhoods of Milan.
- milan1: starting from the shapefile of Milan's map (available in the Datasets folder), divided in NIL (local areas), I added the values of rents from Immobiliare for further work.
