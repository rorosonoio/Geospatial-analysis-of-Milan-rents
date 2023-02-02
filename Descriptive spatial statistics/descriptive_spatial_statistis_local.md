descriptive\_spatial\_statistis\_local
================
Roberta
30/1/2023

The necessary

``` r
library(sf); library(tmap); library(spdep)
```

    ## Warning: il pacchetto 'sf' è stato creato con R versione 4.1.3

    ## Linking to GEOS 3.10.2, GDAL 3.4.1, PROJ 7.2.1; sf_use_s2() is TRUE

    ## Warning: il pacchetto 'tmap' è stato creato con R versione 4.1.3

    ## Warning: il pacchetto 'spdep' è stato creato con R versione 4.1.3

    ## Caricamento del pacchetto richiesto: sp

    ## Warning: il pacchetto 'sp' è stato creato con R versione 4.1.3

    ## Caricamento del pacchetto richiesto: spData

    ## Warning: il pacchetto 'spData' è stato creato con R versione 4.1.3

    ## To access larger datasets in this package, install the spDataLarge
    ## package with: `install.packages('spDataLarge',
    ## repos='https://nowosad.github.io/drat/', type='source')`

``` r
library(spatialreg)
```

    ## Warning: il pacchetto 'spatialreg' è stato creato con R versione 4.1.3

    ## Caricamento del pacchetto richiesto: Matrix

    ## 
    ## Caricamento pacchetto: 'spatialreg'

    ## I seguenti oggetti sono mascherati da 'package:spdep':
    ## 
    ##     get.ClusterOption, get.coresOption, get.mcOption,
    ##     get.VerboseOption, get.ZeroPolicyOption, set.ClusterOption,
    ##     set.coresOption, set.mcOption, set.VerboseOption,
    ##     set.ZeroPolicyOption

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

``` r
coords <- st_centroid(st_geometry(MI))
dnb1772 <- dnearneigh(coords, 0, 1772)
dnb1772.listw <- nb2listw(dnb1772,style="W",zero.policy=F)
mplot <- moran.plot(MI$Rent, listw=dnb1772.listw, main="Moran scatterplot")
grid()
```

![](descriptive_spatial_statistis_local_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->

``` r
MI$hat_value <- mplot$hat
tm_shape(MI) + tm_polygons("hat_value")  
```

![](descriptive_spatial_statistis_local_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->

``` r
mplot <- moran.plot(MI$Rent, listw=dnb1772.listw, main="Moran scatterplot", 
         return_df=F)
```

![](descriptive_spatial_statistis_local_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->

``` r
hotspot <- as.numeric(row.names(as.data.frame(summary(mplot)))) 
```

    ## Potentially influential observations of
    ##   lm(formula = wx ~ x) :
    ## 
    ##    dfb.1_ dfb.x dffit   cov.r   cook.d hat    
    ## 34  0.48  -0.51 -0.53_*  1.15_*  0.14   0.14_*
    ## 35  0.76  -0.80 -0.84_*  1.08_*  0.34   0.14_*
    ## 37 -0.24   0.28  0.39    0.91_*  0.07   0.02  
    ## 56 -0.23   0.28  0.44    0.84_*  0.09   0.02  
    ## 57 -0.36   0.42  0.56_*  0.81_*  0.14   0.03  
    ## 64 -0.10   0.06 -0.27    0.91_*  0.03   0.01  
    ## 88  0.67  -0.71 -0.75_*  1.05    0.27   0.12_*

``` r
MI$wx <- lag.listw(dnb1772.listw, MI$Rent)
```

``` r
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
    ##   HH   LL None 
    ##    6    1   81

``` r
tm_shape(MI) + tm_polygons("quadrant")
```

![](descriptive_spatial_statistis_local_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

``` r
lmI <- localmoran(MI$Rent, dnb1772.listw)

MI$lmI <- lmI[,1]
tm_shape(MI) + 
    tm_polygons("lmI", title = "Local Moran's I values") 
```

    ## Variable(s) "lmI" contains positive and negative values, so midpoint is set to 0. Set midpoint = NA to show the full spectrum of the color palette.

![](descriptive_spatial_statistis_local_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->

``` r
MI$locmpv <- p.adjust(lmI[, "Pr(z != E(Ii))"], "bonferroni")
tm_shape(MI) + 
    tm_polygons("locmpv", title = "Local Moran's I significance map",
                breaks=c(0, 0.0005, 0.001, 0.005, 0.01, 0.05, 0.1, 0.2, 0.5, 0.75, 1)) 
```

![](descriptive_spatial_statistis_local_files/figure-gfm/unnamed-chunk-8-1.png)<!-- -->

``` r
lmIp <- localmoran_perm(MI$Rent, dnb1772.listw, nsim = 9999, iseed = 1) 
MI$locmpvPerm <- p.adjust(lmIp[, "Pr(z != E(Ii)) Sim"], "bonferroni")
tm_shape(MI) + 
    tm_polygons("locmpvPerm", title = "Local Moran's I significance map",
                breaks=c(0, 0.0005, 0.001, 0.005, 0.01, 0.05, 0.1, 0.2, 0.5, 0.75, 1)) 
```

![](descriptive_spatial_statistis_local_files/figure-gfm/unnamed-chunk-9-1.png)<!-- -->

``` r
MI$quadrant2 <- attr(lmI, "quadr")$mean
MI$quadrant2[MI$locmpv>0.005] <- NA
tm_shape(MI) + tm_polygons("quadrant2")
```

![](descriptive_spatial_statistis_local_files/figure-gfm/unnamed-chunk-10-1.png)<!-- -->
