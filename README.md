
pollutedRoutes
==============

Identification of travel routes commonly walked and cycled for air pollution research

Download route data
-------------------

There is a large open dataset on travel to work patterns provided by WICID.

A pre-processed and geographically specific version of this dataset was made available by the Propensity to Cycle Tool (PCT) project (Lovelace et al. 2016).

This can be downloaded, and subset to Leeds as follows:

``` r
url_lines = "https://github.com/npct/pct-data/raw/master/west-yorkshire/l.Rds"
download.file(url = url_lines, destfile = "l.Rds")
l = readRDS("l.Rds")
most_walked = head(order(l$foot, decreasing = TRUE), n = 5)
```

    ## Loading required package: sp

``` r
l_most_walked = l[most_walked,]
plot(l_most_walked)
```

![](README_files/figure-markdown_github/unnamed-chunk-1-1.png)

Let's save that into a language-neutral open geographic file format:

``` r
geojsonio::geojson_write(input = l_most_walked, file = "l_most_walked.geojson")
```

    ## Success! File is at l_most_walked.geojson

    ## <geojson>
    ##   Path:       l_most_walked.geojson
    ##   From class: SpatialLinesDataFrame

We can plot this as an interactive map as follows:

``` r
library(leaflet)
leaflet() %>%
  addTiles() %>% 
  addPolylines(data = l_most_walked)
```

![](README_files/figure-markdown_github/unnamed-chunk-3-1.png)

How many trips are represented in that data?

``` r
sum(l_most_walked$all)
```

    ## [1] 5350

Wow, 5000 trips in just 5 routes: that's a LOT of travel into the centre of Leeds from 5 of the clostest MSOA zones (average population: ~7000).

How many of those are walked?

``` r
sum(l_most_walked$foot)
```

    ## [1] 3747

Over half of them!

That should not be a surprise as the distances involved are so short, averaging around 1.5 km straight-line distance:

``` r
library(stplanr)
line_length(l_most_walked) # results in m
```

    ## Transforming to CRS +proj=aeqd +lat_0=53.79492948 +lon_0=-1.54407447 +x_0=0 +y_0=0 +ellps=WGS84

    ## Running function on a temporary projected version of the Spatial object using the CRS: +proj=aeqd +lat_0=53.79492948 +lon_0=-1.54407447 +x_0=0 +y_0=0 +ellps=WGS84

    ##    50598    50464    74933    50347    50668 
    ## 1671.433 1257.356 2069.989 1303.172 1939.532

Route allocation
----------------

We can allocate these routes to the travel network using a routing service such as GraphHopper:

``` r
r_most_walked = line2route(l = l_most_walked, route_fun = route_graphhopper, vehicle = "foot")
plot(r_most_walked)
```

![](README_files/figure-markdown_github/unnamed-chunk-7-1.png)

``` r
geojsonio::geojson_write(input = r_most_walked, file = "r_most_walked.geojson")
```

    ## Success! File is at r_most_walked.geojson

    ## <geojson>
    ##   Path:       r_most_walked.geojson
    ##   From class: SpatialLinesDataFrame

We can look at the buffer of likely walked routes now, as follows:

``` r
r_buff = buff_geo(shp = r_most_walked, width = 200) # 200 m buffer
```

    ## Transforming to CRS +proj=aeqd +lat_0=53.79448497 +lon_0=-1.5434621 +x_0=0 +y_0=0 +ellps=WGS84

    ## Running function on a temporary projected version of the Spatial object using the CRS: +proj=aeqd +lat_0=53.79448497 +lon_0=-1.5434621 +x_0=0 +y_0=0 +ellps=WGS84

``` r
plot(r_buff)
```

![](README_files/figure-markdown_github/unnamed-chunk-8-1.png)

That represents corridors with high numbers of walkers commuting who may be vulnerable to air polution.

Let's save the result and get on with the challenge!

``` r
geojsonio::geojson_write(input = r_buff, file = "r_buf.geojson")
```

    ## Success! File is at r_buf.geojson

    ## <geojson>
    ##   Path:       r_buf.geojson
    ##   From class: SpatialPolygons

References
----------

Lovelace, R., Goodman, A., Aldred, R., Berkoff, N., Abbas, A., Woodcock, J., 2016. The Propensity to Cycle Tool: An open source online system for sustainable transport planning. Journal of Transport and Land Use 10. <doi:10.5198/jtlu.2016.862>
