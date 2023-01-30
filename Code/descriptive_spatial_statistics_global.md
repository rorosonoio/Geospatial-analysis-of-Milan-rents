descriptive\_spatial\_statistics\_global
================
Roberta
30/1/2023

\#Descriptive spatial statistics - global case

First the necessary libraries are installed:

``` r
library(sf); library(tmap); library(spdep)
```

    ## Warning: il pacchetto 'sf' è stato creato con R versione 4.1.3

    ## Warning: il pacchetto 'tmap' è stato creato con R versione 4.1.3

    ## Warning: il pacchetto 'spdep' è stato creato con R versione 4.1.3

    ## Warning: il pacchetto 'sp' è stato creato con R versione 4.1.3

    ## Warning: il pacchetto 'spData' è stato creato con R versione 4.1.3

``` r
library(spatialreg)
```

    ## Warning: il pacchetto 'spatialreg' è stato creato con R versione 4.1.3

Then, the data is retrieved. In this case it is the shapefile of the
city of Milan, divided in neighborhoods (NIL - Nucleo d’Identità
Locale), which can be downloaded from the folder “Data” of this
repository. Then, in order to have a reference to compute neigborhoods,
the centroids of each area are computed.

``` r
MI <- st_read("milano")
```

    ## Reading layer `milano_map' from data source 
    ##   `C:\Users\rober\Desktop\vaffanculo\milano' using driver `ESRI Shapefile'
    ## Simple feature collection with 88 features and 4 fields
    ## Geometry type: POLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 503174.7 ymin: 5025932 xmax: 521722.7 ymax: 5042508
    ## Projected CRS: RDN2008 / UTM zone 32N (N-E)

``` r
coords <- st_centroid(st_geometry(MI))
```

In order to apply Critical cut-off neighborhood, find the minimum
distance that can ensure that all centroids have at least one
neighborhood.

``` r
knn1IT <- knn2nb(knearneigh(coords,k=1))
all.linkedT <- max(unlist(nbdists(knn1IT, coords))) 
all.linkedT
```

    ## [1] 1771.507

The minimal distance is of 1771.5, so I will use 1772.

The neigborhoods are computed following the method of Critical cut-off.
Then, the weight matrix is computed for further calculations.

``` r
dnb1772 <- dnearneigh(coords, 0, 1772)
dnb2500 <- dnearneigh(coords, 0, 2500)
dnb3000 <- dnearneigh(coords, 0, 3000)

dnb1772.listw <- nb2listw(dnb1772,style="W")
dnb2500.listw <- nb2listw(dnb2500,style="W")
dnb3000.listw <- nb2listw(dnb3000,style="W")
```

``` r
#plot(MI[c("Rent")], main="Rent per square meter in Milan in 2022", nbreaks = 20)
```

``` r
moran.test(MI$Rent, dnb1772.listw, randomisation=FALSE)
```

    ## 
    ##  Moran I test under normality
    ## 
    ## data:  MI$Rent  
    ## weights: dnb1772.listw    
    ## 
    ## Moran I statistic standard deviate = 8.1429, p-value < 2.2e-16
    ## alternative hypothesis: greater
    ## sample estimates:
    ## Moran I statistic       Expectation          Variance 
    ##       0.680207631      -0.011494253       0.007215668

``` r
moran.test(MI$Rent, dnb2500.listw, randomisation=FALSE)
```

    ## 
    ##  Moran I test under normality
    ## 
    ## data:  MI$Rent  
    ## weights: dnb2500.listw    
    ## 
    ## Moran I statistic standard deviate = 10.874, p-value < 2.2e-16
    ## alternative hypothesis: greater
    ## sample estimates:
    ## Moran I statistic       Expectation          Variance 
    ##       0.600292905      -0.011494253       0.003165595

``` r
moran.test(MI$Rent, dnb3000.listw, randomisation=FALSE)
```

    ## 
    ##  Moran I test under normality
    ## 
    ## data:  MI$Rent  
    ## weights: dnb3000.listw    
    ## 
    ## Moran I statistic standard deviate = 11.803, p-value < 2.2e-16
    ## alternative hypothesis: greater
    ## sample estimates:
    ## Moran I statistic       Expectation          Variance 
    ##        0.53266578       -0.01149425        0.00212552

``` r
moran.test(MI$Rent, dnb1772.listw, randomisation=TRUE)
```

    ## 
    ##  Moran I test under randomisation
    ## 
    ## data:  MI$Rent  
    ## weights: dnb1772.listw    
    ## 
    ## Moran I statistic standard deviate = 8.2399, p-value < 2.2e-16
    ## alternative hypothesis: greater
    ## sample estimates:
    ## Moran I statistic       Expectation          Variance 
    ##       0.680207631      -0.011494253       0.007046764

``` r
moran.test(MI$Rent, dnb2500.listw, randomisation=TRUE)
```

    ## 
    ##  Moran I test under randomisation
    ## 
    ## data:  MI$Rent  
    ## weights: dnb2500.listw    
    ## 
    ## Moran I statistic standard deviate = 11.003, p-value < 2.2e-16
    ## alternative hypothesis: greater
    ## sample estimates:
    ## Moran I statistic       Expectation          Variance 
    ##       0.600292905      -0.011494253       0.003091481

``` r
moran.test(MI$Rent, dnb3000.listw, randomisation=TRUE)
```

    ## 
    ##  Moran I test under randomisation
    ## 
    ## data:  MI$Rent  
    ## weights: dnb3000.listw    
    ## 
    ## Moran I statistic standard deviate = 11.944, p-value < 2.2e-16
    ## alternative hypothesis: greater
    ## sample estimates:
    ## Moran I statistic       Expectation          Variance 
    ##       0.532665777      -0.011494253       0.002075813
