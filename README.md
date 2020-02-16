---
title: "osmdata, an R package for OpenStreetMap data"
keywords: "open street map, openstreetmap, overpass API, OSM"
output:
  rmarkdown::html_vignette:
    self_contained: no

  md_document:
    variant: markdown_github
---

<!-- README.md is generated from README.Rmd. Please edit that file -->




# osmdata <a href='https://docs.ropensci.org/osmdata/'><img src='man/figures/osmhex.png' align="right" height=210 width=182/></a>



[![Build Status](https://travis-ci.org/ropensci/osmdata.svg?branch=master)](https://travis-ci.org/ropensci/osmdata) 
[![Build status](https://ci.appveyor.com/api/projects/status/github/ropensci/osmdata?svg=true)](https://ci.appveyor.com/project/ropensci/osmdata)
[![codecov](https://codecov.io/gh/ropensci/osmdata/branch/master/graph/badge.svg)](https://codecov.io/gh/ropensci/osmdata)
[![CRAN_Status_Badge](http://www.r-pkg.org/badges/version/osmdata)](http://cran.r-project.org/web/packages/osmdata) 
[![CRAN Downloads](http://cranlogs.r-pkg.org/badges/grand-total/osmdata?color=orange)](http://cran.r-project.org/package=osmdata)
[![Project Status: Active](http://www.repostatus.org/badges/latest/active.svg)](http://www.repostatus.org/#active)

<!--![](./man/figures/title.png)-->

[![](https://badges.ropensci.org/103_status.svg)](https://github.com/ropensci/onboarding/issues/103)
[![status](http://joss.theoj.org/papers/0f59fb7eaeb2004ea510d38c00051dd3/status.svg)](http://joss.theoj.org/papers/0f59fb7eaeb2004ea510d38c00051dd3)


`osmdata` is an R package for accessing the data underlying
OpenStreetMap (OSM), delivered via the [Overpass
API](https://wiki.openstreetmap.org/wiki/Overpass_API).  (Other packages such as
[`OpenStreetMap`](https://cran.r-project.org/web/packages/OpenStreetMap/index.html)
can be used to download raster tiles based on OSM data.)
[Overpass](https://overpass-turbo.eu) is a read-only API that extracts custom
selected parts of OSM data. Data can be returned in a variety of formats,
including as [Simple Features (`sf`)](https://cran.r-project.org/package=sf),
[Spatial (`sp`)](https://cran.r-project.org/package=sp), or [Silicate
(`sc`)](https://github.com/hypertidy/silicate) objects.
The package is designed to allow access to small-to-medium-sized OSM datasets (see [`geofabrik`](https://github.com/ITSLeeds/geofabrik) for an approach for reading-in bulk OSM data extracts).


## Installation

To install:

```r
# Install from CRAN 
install.packages("osmdata")

# Alternatively, install the development version
# install.packages("remotes")
remotes::install_github("ropensci/osmdata")
```

To load the package and check the version:

```r
library(osmdata)
packageVersion("osmdata")
#> [1] '0.1.3.1'
```

## Usage

[Overpass API](https://wiki.openstreetmap.org/wiki/Overpass_API) queries can be
built from a base query constructed with `opq` followed by `add_osm_feature`. The
corresponding OSM objects are then downloaded and converted to [Simple
Feature (`sf`)](https://cran.r-project.org/package=sf) objects with
`osmdata_sf()`, [Spatial (`sp`)](https://cran.r-project.org/package=sp)
objects with `osmdata_sp()` or [Silicate (`sc`)](https://github.com/hypertidy/silicate)
objects with `osmdata_sc()`.  For example, 


```r
x <- opq(bbox = c(-0.27, 51.47, -0.20, 51.50)) %>% # Chiswick Eyot in London, U.K.
    add_osm_feature(key = 'name', value = "Thames", value_exact = FALSE) %>%
    osmdata_sf()
x
```

```
#> Object of class 'osmdata' with:
#>                  $bbox : 51.47,-0.27,51.5,-0.2
#>         $overpass_call : The call submitted to the overpass API
#>                  $meta : metadata including timestamp and version numbers
#>            $osm_points : 'sf' Simple Features Collection with 24548 points
#>             $osm_lines : 'sf' Simple Features Collection with 2219 linestrings
#>          $osm_polygons : 'sf' Simple Features Collection with 33 polygons
#>        $osm_multilines : 'sf' Simple Features Collection with 6 multilinestrings
#>     $osm_multipolygons : 'sf' Simple Features Collection with 3 multipolygons
```

OSM data can also be downloaded in OSM XML format with `osmdata_xml()` and saved
for use with other software.

```r
osmdata_xml(q1, "data.osm")
```

### Bounding Boxes

All `osmdata` queries begin with a bounding box defining the area of the query.
The [`getbb()`
function](https://docs.ropensci.org/osmdata/reference/getbb.html) can be used
to extract bounding boxes for specified place names.

```r
getbb ("astana kazakhstan")
#>        min      max
#> x 71.22444 71.78519
#> y 51.00068 51.35111
```
The next step is to convert that to an overpass query object with the [`opq()`
function](https://docs.ropensci.org/osmdata/reference/opq.html):

```r
q <- opq (getbb ("astana kazakhstan"))
q <- opq ("astana kazakhstan") # identical result
```
It is also possible to use bounding polygons rather than rectangular boxes:

```r
b <- getbb ("bangalore", format_out = "polygon")
class (b); head (b [[1]])
#> [1] "matrix"
#> [1] 77.4601
```

### Features

The next step is to define features of interest using the [`add_osm_feature()`
function](https://docs.ropensci.org/osmdata/reference/add_osm_feature.html).
This function accepts `key` and `value` parameters specifying desired features
in the [OSM key-vale schema](https://wiki.openstreetmap.org/wiki/Map_Features).
Multiple `add_osm_feature()` calls may be combined as illustrated below, with
the result being a logical AND operation, thus returning all amenities that
are lebelled both as restaurants and also as pubs:

```r
q <- opq ("portsmouth usa") %>%
    add_osm_feature(key = "amenity", value = "restaurant") %>%
    add_osm_feature(key = "amenity", value = "pub") # There are none of these
```
(Logical OR combinations are demonstrated [below](#additional).) Negation can also be specified
by pre-prending an exclamation mark so that the following requests all amenities
that are NOT labelled as restaurants and that are not labelled as pubs:

```r
q <- opq ("portsmouth usa") %>%
    add_osm_feature(key = "amenity", value = "!restaurant") %>%
    add_osm_feature(key = "amenity", value = "!pub") # There are a lot of these
```

Additional arguments allow for more refined matching, such as the following
requrest for all pubs with "irish" in the name:

```r
q <- opq ("washington dc") %>%
    add_osm_feature(key = "amenity", value = "pub") %>%
    add_osm_feature(key = "name", value = "irish",
                    value_exact = FALSE, match_case = FALSE)
```
See
[`?available_features`](https://docs.ropensci.org/osmdata/reference/available_features.html)
and
[`?available_tags`](https://docs.ropensci.org/osmdata/reference/available_tags.html)
for further information.


### Data Formats

An overpass query constructed with the `opq()` and `add_osm_feature()`
functions is then sent to the [overpass server](https://overpass-turbo.eu) to
request data. These data may be returned in a variety of formats, currently
including:

1. XML data (downloaded locally) via
   [`osmdata_xml()`](https://docs.ropensci.org/osmdata/reference/osmdata_xml.html);
2. [Simple Features (sf)](https://cran.r-project.org/package=sf) format via
   [`osmdata_sf()`](https://docs.ropensci.org/osmdata/reference/osmdata_sf.html);
3. [R Spatial (sp)](https://cran.r-project.org/package=sp) format via
   [`osmdata_sp()`](https://docs.ropensci.org/osmdata/reference/osmdata_sp.html);
   and
4. [Silicate (SC)](https://github.com/hypertidy/silicate) format via
   [`osmdata_sc()`](https://docs.ropensci.org/osmdata/reference/osmdata_sc.html).


### Additional Functionality {#additional}

Logical OR combinations can be implemented with the package's internal `c`
method, so that the above example can be extended to all amenities that are
either restaurants OR pubs with

```r
pubs <- opq ("portsmouth usa") %>%
    add_osm_feature(key = "amenity", value = "pub") %>%
    osmdata_sf()
restaurants <- opq ("portsmouth usa") %>%
    add_osm_feature(key = "amenity", value = "restaurant") %>%
    osmdata_sf()
c (pubs, restaurants)
```

```
#> Object of class 'osmdata' with:
#>                  $bbox : 43.0135509,-70.8229994,43.0996118,-70.7279298
#>         $overpass_call : The call submitted to the overpass API
#>                  $meta : metadata including timestamp and version numbers
#>            $osm_points : 'sf' Simple Features Collection with 325 points
#>             $osm_lines : NULL
#>          $osm_polygons : 'sf' Simple Features Collection with 24 polygons
#>        $osm_multilines : NULL
#>     $osm_multipolygons : NULL
```


Data may also be trimmed to within a defined polygonal shape with the
[`trim_osmdata()`](https://docs.ropensci.org/osmdata/reference/trim_osmdata.html)
function.  Full package functionality is described on the
[website](https://docs.ropensci.org/osmdata/)


## Citation


```r
citation ("osmdata")
#> 
#> To cite osmdata in publications use:
#> 
#>   Mark Padgham, Bob Rudis, Robin Lovelace, Maëlle Salmon (2017). osmdata Journal of Open Source
#>   Software, 2(14). URL https://doi.org/10.21105/joss.00305
#> 
#> A BibTeX entry for LaTeX users is
#> 
#>   @Article{,
#>     title = {osmdata},
#>     author = {Mark Padgham and Bob Rudis and Robin Lovelace and Maëlle Salmon},
#>     journal = {The Journal of Open Source Software},
#>     year = {2017},
#>     volume = {2},
#>     number = {14},
#>     month = {jun},
#>     publisher = {The Open Journal},
#>     url = {https://doi.org/10.21105/joss.00305},
#>     doi = {10.21105/joss.00305},
#>   }
```

## Code of Conduct

Please note that this project is released with a [Contributor Code of
Conduct](CODE_OF_CONDUCT.md).  By participating in this project you agree to
abide by its terms.

[![ropensci\_footer](https://ropensci.org/public_images/github_footer.png)](https://ropensci.org)
