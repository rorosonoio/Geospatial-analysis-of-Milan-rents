# Descriptive spatial statistics in the global case

First the necessary libraries are installed:

``` r
library(sf); library(tmap); library(spdep)
library(spatialreg)
```

Then, the data is retrieved. In this case it is the shapefile of the
city of Milan, divided in neighborhoods (NIL - Nucleo d’Identità
Locale), which can be downloaded from the folder “Data” of this
repository. Then, in order to have a reference to compute neighborhoods,
the centroids of each area are computed.

``` r
MI <- st_read("milano")
```

    ## Reading layer `milano_map' from data source 
    ##   `C:\Users\rober\Desktop\project\milano' using driver `ESRI Shapefile'
    ## Simple feature collection with 88 features and 4 fields
    ## Geometry type: POLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: 503174.7 ymin: 5025932 xmax: 521722.7 ymax: 5042508
    ## Projected CRS: RDN2008 / UTM zone 32N (N-E)

``` r
coords <- st_centroid(st_geometry(MI))
```

In order to apply Critical cut-off neighborhood, it is needed to find
the minimum distance that can ensure that all centroids have at least
one neighborhood.

``` r
knn1IT <- knn2nb(knearneigh(coords,k=1))
all.linkedT <- max(unlist(nbdists(knn1IT, coords))) 
all.linkedT
```

    ## [1] 1771.507

The minimal distance is of 1771.5, so I will use 1772.

The neighborhoods are computed following the method of Critical cut-off.
Then, the weight matrix is computed for further calculations.

``` r
dnb1772 <- dnearneigh(coords, 0, 1772)

dnb1772.listw <- nb2listw(dnb1772,style="W")
```

# Visual inspection

It can be seen from the plot that the higher prices seem to be clustered
in the center of the city, while the prices lower the further the area
is from the center. This does not suggest a random or sparse behavior of
the rent prices, however, the Moran I test can confirm objectively if
areas with high prices influence the nearby areas as well.

![Plot](Descriptive spatial statistics/Plots and figures/unnamed-chunk-4-1.png)

# Moran I test

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

The test was performed under the assumption of normality. It can be seen
that the Moran I statistic value is high, which suggests that if the
value of the rent in an area increases, the neighboring areas will be
influenced positively. The p-value is low (&lt;0.01), which mean that we
reject the null hypothesis, which stated that there was no spatial
dependence between the rent values of the neighborhoods of Milan.

The test can also be performed under the assumption of randomization of
the data.

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

In this case as well, it is confirmed that the null hypothesis is
rejected, which means that the rent price of a neighborhood influences
positively the neighboring areas too.
