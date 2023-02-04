# Descriptive local statistics

The necessary packages and data is imported.

``` r
library(sf); library(tmap); library(spdep)
library(spatialreg)
```

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

This time, instead of computing the global spatial dependence, the focus
moves to local spatial dependence. The aim is to identify local clusters
which influence the most the surrounding area.

As a first thing the Moran scatterplot is computed.

``` r
coords <- st_centroid(st_geometry(MI))
dnb1772 <- dnearneigh(coords, 0, 1772)
dnb1772.listw <- nb2listw(dnb1772,style="W",zero.policy=F)
mplot <- moran.plot(MI$Rent, listw=dnb1772.listw, main="Moran scatterplot")
grid()
```

![](https://github.com/rorosonoio/Geospatial-analysis-of-Milan-rents/blob/dc00d438885ea5ed785c413ec657ee7d5a6832dd/Descriptive%20spatial%20statistics/Plots%20and%20figures/unnamed-chunk-3-1.png)<!-- -->

It can be noticed that no influential region are present in the areas of
High-High and Low-Low, which suggests that no area of big influence is
negatively correlated to the others. However, some values fall into the
High-Low, but not highly influential ones.

``` r
hotspot <- as.numeric(row.names(as.data.frame(summary(mplot)))) 
MI$wx <- lag.listw(dnb1772.listw, MI$Rent)
MI$quadrant <- rep("None", length(MI$Rent))
for(i in 1:length(hotspot))  {
  if (MI$Rent[hotspot[i]]>mean(MI$Rent) & MI$wx[hotspot[i]]> mean(MI$wx)) 
        MI$quadrant[hotspot[i]] <- "HH" 
  if (MI$Rent[hotspot[i]]>mean(MI$Rent) & MI$wx[hotspot[i]]< mean(MI$wx)) 
        MI$quadrant[hotspot[i]] <- "HL" 
  if (MI$Rent[hotspot[i]]<mean(MI$Rent) & MI$wx[hotspot[i]]<mean(MI$wx)) 
        MI$quadrant[hotspot[i]] <- "LL" 
  if (MI$Rent[hotspot[i]]<mean(MI$Rent) & MI$wx[hotspot[i]]>mean(MI$wx)) 
        MI$quadrant[hotspot[i]] <- "LH" 
  }
table(MI$quadrant)
```

    ## 
    ##   HH   HL   LH   LL None 
    ##   15    1   10   34   28

``` r
tm_shape(MI) + tm_polygons("quadrant")
```

![](https://github.com/rorosonoio/Geospatial-analysis-of-Milan-rents/blob/dc00d438885ea5ed785c413ec657ee7d5a6832dd/Descriptive%20spatial%20statistics/Plots%20and%20figures/unnamed-chunk-5-1.png)<!-- -->

``` r
lmI <- localmoran(MI$Rent, dnb1772.listw)

MI$lmI <- lmI[,1]
tm_shape(MI) + 
    tm_polygons("lmI", title = "Local Moran's I values") 
```

    ## Variable(s) "lmI" contains positive and negative values, so midpoint is set to 0. Set midpoint = NA to show the full spectrum of the color palette.

![](https://github.com/rorosonoio/Geospatial-analysis-of-Milan-rents/blob/dc00d438885ea5ed785c413ec657ee7d5a6832dd/Descriptive%20spatial%20statistics/Plots%20and%20figures/unnamed-chunk-6-1.png)<!-- -->

It can be noticed that the values of some neighborhoods in the western
part of the city seem to have a high Moran’s I value. It can also be
noticed, from the choropleth present in the global analysis, that those
areas had the lowest prices in rent. This could mean that being this far
away from the center makes it an influential area, and it could
influence the nearby areas to have lower prices than the city center.

Given the previous calculation, it is useful to calculate the actual
significance of those result.

``` r
MI$locmpv <- p.adjust(lmI[, "Pr(z != E(Ii))"], "bonferroni")
tm_shape(MI) + 
    tm_polygons("locmpv", title = "Local Moran's I significance map",
                breaks=c(0, 0.0005, 0.001, 0.005, 0.01, 0.05, 0.1, 0.2, 0.5, 0.75, 1)) 
```

![](https://github.com/rorosonoio/Geospatial-analysis-of-Milan-rents/blob/dc00d438885ea5ed785c413ec657ee7d5a6832dd/Descriptive%20spatial%20statistics/Plots%20and%20figures/unnamed-chunk-7-1.png)<!-- -->

As it can be noticed, the most relevant results are the ones of the
areas in the city center, which are the most influential and cause the
biggest rise in rents, were they to rise prices themselves. The other
areas, despite obtaining a higher Moran’s I value, do not seem to have a
significant p-value, which means that those results are not significant
in our analysis and they do not hold influence on other areas.
