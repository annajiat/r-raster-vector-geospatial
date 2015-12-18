---
layout: post
title: "Lesson 05: Raster Time Series Data in R"
date:   2015-10-24
authors: [Jason Williams, Jeff Hollister, Kristina Riemer, Mike Smorul, Zack
Brym, Leah Wasser]
contributors: [Megan A. Jones]
packagesLibraries: [raster, rgdal, rasterVis]
dateCreated:  2014-11-26
lastModified: 2015-12-17
category: time-series-workshop
tags: [raster-ts-wrksp, raster]
mainTag: raster-ts-wrksp
description: "This lesson covers how to work with a raster time series, using
an `R` RasterStack object. It also covers practical assessment of data quality
in remote sensing derived imagery. Finally, it covers plotting raster time
series."
code1: SR05-Time-Series-Raster-In-R.R
image:
  feature: NEONCarpentryHeader_2.png
  credit: A collaboration between the National Ecological Observatory Network (NEON) and Data Carpentry
  creditlink: http://www.neoninc.org
permalink: /R/Raster-Times-Series-Data-In-R/
comments: false
---

{% include _toc.html %}

##About
This lesson covers how to work with a raster time series, using an `R`
RasterStack object. It also covers practical assessment of data quality
in remote sensing derived imagery. Finally, it covers simple plotting of raster
time series.

**R Skill Level:** Intermediate - you've got the basics of `R` down.

<div id="objectives" markdown="1">

###Goals / Objectives

After completing this activity, you will:

* Understand what time series raster format is.
* Know how to work with a set of time series rasters. 
* Be able to efficiently import a set of rasters in a single directory.
* Be able to plot and explore time series raster data using the `plot()`
function.

###Challenge Code
Throughout the lesson we have Challenges that reinforce learned skills. Possible
solutions to the challenges are not posted on this page, however, the code for 
each challenge is in the `R` code that can be downloaded for this lesson (see 
footer on this page).

###Things You'll Need To Complete This Lesson
Please be sure you have the most current version of `R` and, preferably,
RStudio to write your code.

####R Libraries to Install:

* **raster:** `install.packages("raster")`
* **rgdal:** `install.packages("rgdal")`

####Data to Download

<a href=" https://ndownloader.figshare.com/files/3579867" class="btn btn-success"> Download Landsat derived NDVI raster files</a> 

The imagery data used to create this raster teaching data subset were collected
over the National Ecological Observatory Network's
<a href="http://www.neoninc.org/science-design/field-sites/harvard-forest" target="_blank" >Harvard Forest</a>
and 
<a href="http://www.neoninc.org/science-design/field-sites/san-joaquin-experimental-range" target="_blank" >San Joaquin Experimental Range</a>
field sites.  
The imagery was created by the U.S. Geological Survey (USGS) using a 
<a href="http://eros.usgs.gov/#/Find_Data/Products_and_Data_Available/MSS" target="_blank" >  multispectral scanner</a>
on a <a href="http://landsat.usgs.gov" target="_blank" > Landsat Satellite </a>.
The data files are Geographic Tagged Image-File Format (GeoTIFF). 

####Setting the Working Directory
The code in this lesson assumes that you have set your working directory to the
location of the unzipped file of data downloaded above.  If you would like a
refresher on setting the working directory, please view the [Setting A Working Directory In R]({{site.baseurl}}/R/Set-Working-Directory "R Working Directory Lesson") 
lesson prior to beginning this lesson.

###Raster Lesson Series 
This lesson is a part of a series of raster data in R lessons:

* [Lesson 00 - Intro to Raster Data in R]({{ site.baseurl}}/R/Introduction-to-Raster-Data-In-R/)
* [Lesson 01 - Plot Raster Data in R]({{ site.baseurl}}/R/Plot-Rasters-In-R/)
* [Lesson 02 - Reproject Raster Data in R]({{ site.baseurl}}/R/Reproject-Raster-In-R/)
* [Lesson 03 - Raster Calculations in R]({{ site.baseurl}}/R/Raster-Calculations-In-R/)
* [Lesson 04 - Work With Multi-Band Rasters - Images in R]({{ site.baseurl}}/R/Multi-Band-Rasters-In-R/)
* [Lesson 05 - Raster Time Series Data in R]({{ site.baseurl}}/R/Raster-Times-Series-Data-In-R/)
* [Lesson 06 - Plot Raster Time Series Data in R Using RasterVis and LevelPlot]({{ site.baseurl}}/R/Plot-Raster-Times-Series-Data-In-R/)
* [Lesson 07- Extract NDVI Summary Values from a Raster Time Series]({{ site.baseurl}}/R/Extract-NDVI-From-Rasters-In-R/)

###Additional Resources
* <a href="http://cran.r-project.org/web/packages/raster/raster.pdf" target="_blank">
Read more about the `raster` package in `R`.</a>

</div>

#About Raster Time Series Data

Raster data can be in a single band or data layer format. Or it can contains
multiple datasets or bands. If the data are images, each band may represent
reflectance for a different wavelength (color) or set of wavelengths - for
example red, green and blue. Raster data may also contains layers that represent
data collected at different times for the same area and of the same resolution.

<figure>
    <a href="{{ site.baseurl }}/images/raster_timeseries/GreenessOverTime.png">
    <img src="{{ site.baseurl }}/images/raster_timeseries/GreenessOverTime.png"></a>
    <figcaption>A raster dataset can contain a time series. In R, a stack of
    rasters will be in the same extent, CRS and resolution. Image Source: NEON,
    Inc. 
    </figcaption>
</figure>

In this lesson, we are working with a set of rasters that were derived from the 
Landsat satellite and are stored in GeoTIFF format. Each raster covers part of
the  <a href="http://www.neoninc.org/science-design/field-sites/harvard-forest" target="_blank">NEON Harvard Forest field site</a>.

In this lesson, we will

1. Import NDVI data derived from the Landsat Sensor in raster GeoTIFF format
2. Plot one full year of NDVI raster time series data. 
3. Compare RGB imagery with NDVI data to understand strange values in the NDVI. 

##NDVI data
The rasters we will use in this lesson, located in the (`Landsat_NDVI\HARV\ndvi`)
directory, are Normalized Difference Vegetation Index (NDVI) data. NDVI is a
quantitative index of greenness ranging from 0-1 where 0 represents minimal or
no greenness and 1 represents maximum greenness. 

NDVI is often used for a quantative measure of vegetation health, cover and
vegetation phenology (life cycle stage) over large areas. The NDVI data is a Landsat derived single band product saved as a GeoTIFF for different times of the year. 

<figure>
 <a href="http://earthobservatory.nasa.gov/Features/MeasuringVegetation/Images/ndvi_example.jpg"><img src="http://earthobservatory.nasa.gov/Features/MeasuringVegetation/Images/ndvi_example.jpg"></a>
    <figcaption>NDVI is calculated from the visible and near-infrared light
    reflected by vegetation. Healthy vegetation (left) absorbs most of the
    visible light that hits it, and reflects a large portion of the
    near-infrared light. Unhealthy or sparse vegetation (right) reflects more
    visible light and less near-infrared light. Image & Caption Source: NASA </figcaption>
</figure>

* <a href="http://earthobservatory.nasa.gov/Features/MeasuringVegetation/measuring_vegetation_2.php" target="_blank">More on NDVI from NASA</a>

##RGB data
While the NDVI data is a single band product, the RGB images represent 3 (of the 
7) 30m resolution bands available on Landsat. The RGB directory contains RGB
images for each time period that NDVI is available.


<figure>
    <a href="{{ site.baseurl }}/images/raster_timeseries/RGBSTack_1.png">
    <img src="{{ site.baseurl }}/images/raster_timeseries/RGBSTack_1.png"></a>
    <figcaption>A color image consists of 3 bands - red, green and blue. When
    rendered together in a GIS, or even a tool like Photoshop or any other
    image software, they create a color image. Image Source: NEON,Inc.  
    </figcaption>
</figure>


##Getting Started 
In this lesson, we will use the `raster` and `rgdal` packages.


    library(raster)
    library(rgdal)

To begin, we will create a list of raster files that can be used to
generate a `RasterStack`. We can use `list.files()` to designate the list. We
will tell `R` to only find files with a `.tif` extention using the syntax
`pattern=".tif$"`.

If we specify `full.names=TRUE`, the full path for each file will be added to
the list. We will be able to create a `RasterStack` directly from the list.


    # Create list of NDVI file paths
     #assign path to object = cleaner code
    NDVI_HARV_path <- "Landsat_NDVI/HARV/2011/ndvi" 
    all_NDVI_HARV <- list.files(NDVI_HARV_path,
                                full.names = TRUE,
                                pattern = ".tif$")
    
    #view list - note the full path, relative to our working directory, is included
    all_NDVI_HARV

    ##  [1] "Landsat_NDVI/HARV/2011/ndvi/005_HARV_ndvi_crop.tif"
    ##  [2] "Landsat_NDVI/HARV/2011/ndvi/037_HARV_ndvi_crop.tif"
    ##  [3] "Landsat_NDVI/HARV/2011/ndvi/085_HARV_ndvi_crop.tif"
    ##  [4] "Landsat_NDVI/HARV/2011/ndvi/133_HARV_ndvi_crop.tif"
    ##  [5] "Landsat_NDVI/HARV/2011/ndvi/181_HARV_ndvi_crop.tif"
    ##  [6] "Landsat_NDVI/HARV/2011/ndvi/197_HARV_ndvi_crop.tif"
    ##  [7] "Landsat_NDVI/HARV/2011/ndvi/213_HARV_ndvi_crop.tif"
    ##  [8] "Landsat_NDVI/HARV/2011/ndvi/229_HARV_ndvi_crop.tif"
    ##  [9] "Landsat_NDVI/HARV/2011/ndvi/245_HARV_ndvi_crop.tif"
    ## [10] "Landsat_NDVI/HARV/2011/ndvi/261_HARV_ndvi_crop.tif"
    ## [11] "Landsat_NDVI/HARV/2011/ndvi/277_HARV_ndvi_crop.tif"
    ## [12] "Landsat_NDVI/HARV/2011/ndvi/293_HARV_ndvi_crop.tif"
    ## [13] "Landsat_NDVI/HARV/2011/ndvi/309_HARV_ndvi_crop.tif"

Now we have a list of all GeoTIFF files in the `ndvi` directory for Harvard
Forest. Once we have this list, we can create a stack and begin to work with the
data. 


    # Create a time series raster stack
    NDVI_HARV_stack <- stack(all_NDVI_HARV)

We can also explore the GeoTIFF tags (the embedded metadata) to learn more about
key metadata for our data including the Coordinate Reference System (`CRS`), 
`extent` and raster `resolution`.


    #view crs of rasters
    crs(NDVI_HARV_stack)

    ## CRS arguments:
    ##  +proj=utm +zone=19 +ellps=WGS84 +units=m +no_defs

    #view extent of rasters in stack
    extent(NDVI_HARV_stack)

    ## class       : Extent 
    ## xmin        : 239415 
    ## xmax        : 239535 
    ## ymin        : 4714215 
    ## ymax        : 4714365

    #view the y resolution of our rasters
    yres(NDVI_HARV_stack)

    ## [1] 30

    #view the x resolution of our rasters
    xres(NDVI_HARV_stack)

    ## [1] 30

#Challenge: Raster Metadata
Before going further, answer the following questions about our `RasterStack`.

1. What is the `CRS`?
2. What is the `resolution` of the data? And what `units` is that resolution in?



#Plotting Time Series Data
Once we have created our `RasterStack`, we can visualize our data. We can us
the `plot()` command to quickly plot a `RasterStack`.


    #view a plot of all of the rasters
    #'nc' specifies number of columns (we will have 13 plots)
    plot(NDVI_HARV_stack, 
         zlim = c(1500, 10000), 
         nc = 4)

![ ]({{ site.baseurl }}/images/rfigs/05-Time-Series-Raster/plot-time-series-1.png) 

Wait, the accepted values for NDVI range from 0-1. Why does our data range from
0 - 10,000? 

The metadata for this NDVI data specifies a scale factor: 10,000. A scale factor
is sometimes used to remove decimals. Storing decimal places creates larger file
sizes compared to storing integer values.

Note: We could make this plot (`levelplot`) even prettier by fixing the
individual tile names and adding an overall plot title. How to do this is
covered in the NEON Data Skills 
[Plot Time Series Rasters in R ]({{ site.baseurl }}/R/Plot-Raster-Times-Series-Data-In-R/) 
lesson. {: .notice2}

##Take a Closer Look at Our Data
Let's take a close look at the plots of our data. Given what you might know 
about the seasons in Massachusettes where the NEON Harvard Forest Field Site is
located, do you notice anything that seems unusual about the patterns of
greening and browning of the vegetation at the site?  

Hint: the  number after the X in the tile title is the Julian day.  If you are
unfamiliar with Julian day, check out the NEON Data Skills 
[Converting to Julian Day ]({{ site.baseurl }}/R/julian-day-conversion/) 
lesson.

What is another way we can look at these data that is a bit more quantitative 
than viewing images? We could use histograms to explore the distribution of NDVI
values.


    #create histograms of each image
    hist(NDVI_HARV_stack, 
         xlim = c(0, 10000))

![ ]({{ site.baseurl }}/images/rfigs/05-Time-Series-Raster/view-stack-histogram-1.png) 

It seems like things get green in the spring and summer like we expect, but the 
data at Julian days 277 and 293 are unusual. It appears as if the vegetation got
green in the spring, but then died back only to get green again towards the end of the year. Is this right?

###Exploring Unusual Data Patterns
The NDVI data that we are using comes from 2011, perhaps a strong freeze around
Julian day 277 could cause a major plant die off. 

![ ]({{ site.baseurl }}/images/rfigs/05-Time-Series-Raster/view-temp-data-1.png) 

There are no significant peaks or dips in the temperature during the late summer
or early fall time period that could logically cause cause the vegetation to
brown (around day 277) and then quickly green again (by day 309). 

What is happening here? 

The temperature data suggests that the pattern of warming and cooling is fairly
consistent. Maybe we should look at the source data that were used to derive our
NDVI rasters to try to understand what appears to be outlier NDVI values.

#Challenge: Examine RGB Raster Files

1. Load the imagery located in the `RGB` directory. 
2. Plot the RGB images & identify the plots for the Julian days 277 and 293.
3. Does viewing the RGB imagery from these two days help explain the NDVI data?  

![ ]({{ site.baseurl }}/images/rfigs/05-Time-Series-Raster/view-all-rgb-1.png) 

##Explore the Data Source
The third challenge questions, "Does viewing the RGB imagery from these two days
help explain the NDVI data?" highlights the importance of exploring the source
of any data we work with.  

The NDVI values from day 277 to 293 were very low, suggesting vegetation
unhealthy, senescent or dead vegetation.  However, when we look at the RGB
images from which the NDVI data was derived we see that nearly the entire 
image is filled with clouds.  The very low NDVI values resulted from there being
no growing vegetation in the field of view.  