---
title: "R-spatial evolution: retirement of rgdal, rgeos and maptools"
author: "Edzer Pebesma, Roger Bivand"
date:  "Apr 12, 2022"
comments: false
layout: post
categories: r
---
* TOC 
{:toc}

\[[view raw
Rmd](https://raw.githubusercontent.com//r-spatial/r-spatial.org/gh-pages/_rmd/2022-03-09-evolution.Rmd)\]

**Summary**: Packages `rgdal`, `rgeos` and `maptools` will retire by the
end of 2023 . We describe where their functionality will go, what
package maintainers can or should do, and which steps we will take to
minimize the impact on dependent packages and on reproducibility in
general.

Why this blog?
==============

The R Consortium ISC granted a proposal by the authors of this blog,
called “Preparing CRAN for the retirement of rgdal, rgeos and maptools”.
In this blog we will point out what this project will do.

Why `rgeos` and `rgdal` will retire
===================================

When loading package `rgeos` recently you may have noticed the startup
message:

    Please note that rgeos will be retired by the end of 2023,
    plan transition to sf functions using GEOS at your earliest convenience.

and similar with `rgdal`:

    Please note that rgdal will be retired by the end of 2023,
    plan transition to sf/stars/terra functions using GDAL and PROJ
    at your earliest convenience.

The reason that these package will retire are primary that their
maintainer, Roger Bivand, has retired. In principle someone could take
over maintenance, but that would be a large effort better spent
differently. Maintenance of these two packages is hard because

1.  they link R code to external libraries, in particular
    [GEOS](https://trac.osgeo.org/geos), [GDAL](https://gdal.org/) and
    [PROJ](https://proj.org/), which needs regular adaption when these
    libraries change
2.  building binary package versions for Windows and Mac-OS requires
    frequent communication with the CRAN team members
3.  both packages are nearly 20 years old, contain a lot of code that is
    conditional to older versions of the external libraries and use the
    `.Call()` interface which makes code harde to read.

More modern packages like `sf` (since 2016), `stars` (2018) and `terra`
(2020) also link to GDAL, GEOS and PROJ and should be able to take over
the role that `rgdal` and `rgeos` now have in the future. In this blog
we describe how we envision this to happen.

Packages depending on `rgeos` and `rgdal`
=========================================

There are several ways in which R packages can depend on others, which
we summarize with **strong** (one of: “Depends”, “Imports”, “LinkingTo”)
or **most** (one of: “Depends”, “Imports”, “LinkingTo”, “Suggests”). The
following table lists how many packages currently depend on `rgdal`
and/or `rgeos` and/or `maptools`:

<table>
<thead>
<tr class="header">
<th></th>
<th style="text-align: right;">rgdal</th>
<th style="text-align: right;">rgeos</th>
<th style="text-align: right;">maptools</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Direct strong</td>
<td style="text-align: right;">213</td>
<td style="text-align: right;">140</td>
<td style="text-align: right;">93</td>
</tr>
<tr class="even">
<td>Recursive strong</td>
<td style="text-align: right;">265</td>
<td style="text-align: right;">190</td>
<td style="text-align: right;">641</td>
</tr>
<tr class="odd">
<td>Direct most</td>
<td style="text-align: right;">358</td>
<td style="text-align: right;">225</td>
<td style="text-align: right;">180</td>
</tr>
</tbody>
</table>

where “recursive strong” also counts packages that depend strongly on
another package that strongly depends on rgdal or rgeos. Strong
dependency means that a package will not run without rgdal or rgeos.

The Plan
========

The plan is to

1.  Move, where possible, functionality from CRAN packages “rgeos”,
    “rgdal” and “maptools” to modern packages that have an active
    maintainer
2.  Contact maintainers of CRAN packages depending on “rgdal”, “rgeos”
    and “maptools” and provide guidance how they can migrate to modern
    alternatives
3.  Modify package “sp” such that it no longer depends on “rgdal” or
    “rgeos”
4.  Assist in modifying package “raster” such that it no longer needs
    “rgdal” or “rgeos”
5.  Develop proxy packages for “rgdal” and “rgeos” that do not link to
    system requirements GDAL, PROJ and GEOS but that - where possible -
    call into maintained R packages (while warning that deprecated
    functions are being called), and otherwise raise errors with
    pointers to how they can be resolved.

Steps that will need to be carried out include:

-   For each of “rgdal”, “rgeos” and “maptools”, we will create a list
    of functions that reverse CRAN dependencies call, along with the
    arguments that are being set in these calls
-   Given this list, we will decide which functions will/can be replaced
    by functions that call into sf/stars/raster/terra, taking vector
    representations first and raster representations second
-   A reference group will be established (a “project steering
    committee”) consisting of the most affected package maintainers and
    institutional R Spatial users, and those with knowledge of legacy
    sp/raster workflows to explore changes that would work for them
-   Functions/methods that can be moved directly to sp will be
    identified (e.g. sp-spatstat coercion, sp-maps coercion etc.)
-   Other functions will be rewritten into a new set of “proxy” packages
    for rgdal, rgeos and maptools that pass on to sf/stars/raster/terra
    (first vector representations, once vector established to be
    extended to raster representations)
-   With these “proxy” packages, reverse dependency checks will be
    carried out (first vector representations, once vector established
    extend to raster representations)
-   Reverse dependency package maintainers affected will be informed
    that this will happen, and will be pointed to reports/contingency
    measures permitting the elimination of proxy packages other than sp
-   All affected reverse dependency package maintainers will be informed
    about functions that will be abandoned and not replaced
-   Proxy packages will be submitted to CRAN if necessary – target to
    support workflows and notebooks (reproducible research) rather than
    dependent packages – sp to continue as a container for a very few
    proxified functions such as sp::spTransform().
-   Old-style function calls will be finally deprecated (e.g. readOGR),
    with pointers to replacement function calls.

Packages depending on `sp` and `raster`
=======================================

Package `raster` has `rgdal` in `Suggests:`, but uses it intensively to
read and write common raster formats. A lot of functions in `rgdal` were
added there specifically with usage by `raster` in mind. The risk that
`raster` will stop working when `rgdal` retires is however limited,
because (i) package `terra` is being developed and is meant to entirely
replace `raster`, (ii) `raster` currently imports `terra`, and (iii) as
`terra` directly links to GDAL, GEOS and PROJ, functions currently in
`rgdal` but needed (only) by `raster` could relatively easily be moved
to `terra`.

Package `sp` currently suggests both `rgeos` and `rgdal`. It uses
`rgdal` for the validation of `CRS` objects (proj4string or WKT
descriptions of coordinate reference systems), and it uses `rgeos` to
sort out which rings are holes when a `SpatialPolygons` object does not
have a flag indicating this. This only happens for saved objects that
are at least 10 years old. Both functionalities (CRS validation, hole
detection) can however easily be substituted by functions in package
`sf`.

We already carried out step 3 of the above list of steps. Branch
[evolution](https://github.com/rsbivand/sp/tree/evolution) of Roger
Bivand’s fork of `sp`, which can be installed with

    devtools::install_github("rsbivand/sp@evolution")

contains conditional code that prevents `sp` to call code in `rgdal` or
`rgeos`. This can be enabled by setting

    Sys.setenv("_SP_EVOLUTION_STATUS_"=2)

after that, e.g. a call to `CRS("+proj=longlat")` calls `sf::st_crs()`
to validate the coordinate reference system and retrieve the WKT
representation. All code of the [ASDAR](https://asdar-book.org/) has
been verified to run under these conditions. Maintainers of packages
depending on `sp` can do this too when checking their packages.
