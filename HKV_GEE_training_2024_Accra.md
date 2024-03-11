
- [1. Day of Year (DoY) rainfall onset](#1day-of-year-doy-rainfall-onset)
   * [1.1.	Define area and load data](#11define-area-and-load-data)
   * [1.2.	Sum the data and create a mask](#12sum-the-data-and-create-a-mask)
   * [1.3.	Cumulative precipitation in a function](#13cumulative-precipitation-in-a-function)
- [2	Day of Year (DoY) rainfall onset for multiple years](#2day-of-year-doy-rainfall-onset-for-multiple-years)
   * [2.1.	Define area of interest and select values](#21define-area-of-interest-and-select-values)
   * [2.2.	Define a function to calculate DoY for multiple years](#22define-a-function-to-calculate-doy-for-multiple-years)
   * [2.3	Create a collection of images](#23create-a-collection-of-images)
   * [2.4 Store the images](#24store-the-images)
   * [2.5.	Create the images and add them to the map](#25create-the-images-and-add-them-to-the-map)
   * [2.6 Export results](#26export-results)
   * [2.7. Histogram](#27histogram)
   * [2.8. Bonus: visualisation of results in Vega-Altair](#28bonus-visualisation-of-results-in-vega-altair)
- [3. Analysis and visualisation of rainfall onset in GIS](#3analysis-and-visualisation-of-rainfall-onset-in-gis)
   * [Type of GIS software**](#type-of-gis-software)
   * [3.1	Importing results in GIS](#31importing-results-in-gis)
   * [3.2	Add a background](#32add-a-background)
   * [3.3	Visualise the DoY layer](#33visualise-the-doy-layer)
   * [3.4	Explore the information about the layer](#34explore-the-information-about-the-layer)
   * [3.5	Analyse the data in the DoY raster layer](#35analyse-the-data-in-the-doy-raster-layer)
   * [3.6	Sample raster values in point layer](#36sample-raster-values-in-point-layer)
   * [3.7	Analyse the results in Excel](#37analyse-the-results-in-excel)
   * [3.8	Composing maps](#38composing-maps)
- [4.	Day of Year (DoY) flood onset](#4day-of-year-doy-flood-onset)
   * [4.1	Define area and visualise region](#41define-area-and-visualise-region)
   * [4.2	Define ImageCollection](#42define-imagecollection)
   * [4.3	Visualise a SAR Image](#43visualise-a-sar-image)
   * [4.4	Focal Statistics and Morphological Operations Theory](#44focal-statistics-and-morphological-operations-theory)
   * [4.5	Focal Statistics and Morphological Operations Code](#45focal-statistics-and-morphological-operations-code)
   * [4.6	Flood onset Day of Year](#46flood-onset-day-of-year)
   * [4.7	Flood onset Day of Year by Function](#47flood-onset-day-of-year-by-function)
   * [4.8	Apply Function to ImageCollection](#48apply-function-to-imagecollection)
   * [4.9	Reduce ImageCollection to image with first day of year with water](#49reduce-imagecollection-to-image-with-first-day-of-year-with-water)
   * [4.10	Beautified code](#410beautified-code)
- [5.	Do it for multiple years](#5do-it-for-multiple-years)
   * [5.1	Define function to process a single year](#51define-function-to-process-a-single-year)
   * [5.2.	Apply function to multiple years](#52apply-function-to-multiple-years)
   * [5.3.	From list of images to bands in single image](#53from-list-of-images-to-bands-in-single-image)
   * [5.4.	Visualize each band as a single layer](#54visualize-each-band-as-a-single-layer)
   * [5.5.	Export the image as an asset](#55export-the-image-as-an-asset)
   * [5.6.	Everything of chapter 4 & 5 in once](#56everything-of-chapter-4--5-in-once)





# 1. Day of Year (DoY) rainfall onset
In training 1 we already looked into the onset of the rainy season for northern Ghana. We defined the onset of the rainy season as the day on which a total of 50mm rainfall had fallen since the start of the year. To determine the DoY of the rainfall onset, we used CHIRPS daily rainfall data. As a refresh exercise, we will build up this DoY rainfall data again. 

## 1.1.	Define area and load data
We start with a new script. In this script we create a polygon that defines northern Ghana. 
![image](https://gist.github.com/assets/5186265/a71f080d-0132-46e4-b752-49132d177b7a)

Import your polygon as a variable (call it _northern_Ghana_) and center and zoom your map.
Your result might look like this: 
![image](https://gist.github.com/assets/5186265/3af3b472-d555-4e03-8957-ec1f99753ce5)

Next, we read the daily CHIRPS data for 2022 for the selected area. To see what data we imported, we print the result and add the first image to the map. Check the results, for example the number of images in the image collection. Do the results match your expectations?

```js
Map.centerObject(northern_Ghana, 8);

// Image collection using CHIRPS
var im_coll = ee.ImageCollection('UCSB-CHG/CHIRPS/DAILY')
    .filterDate('2022-01-01', '2023-01-01')
    .select(['precipitation'])
    .sort('system:time_start');
    
print(im_coll);
Map.addLayer(im_coll.first());
```

## 1.2.	Sum the data and create a mask
As a first step to analyze the DoY of the rainfall onset, we sum the daily rainfall until a certain date. As an example, we have look at 2022-01-01 up to 2022-04-01. To optimize processing speed, we clip the data to the northern Ghana polygon. 

For this summed and clipped layer we create a mask to detect per pixel if the cumulative precipitation (up to the date we chose) has exceeded the threshold of 50mm or not. Both maps are plotted to the map. 

**Can you create these two maps for another range of dates (for example the 15th of April 2022)? And how about a different threshold (for example 35 mm)?**

```js
Map.centerObject(northern_Ghana, 8);

// Image collection using CHIRPS
var im_coll = ee.ImageCollection('UCSB-CHG/CHIRPS/DAILY')
    .filterDate('2022-01-01', '2023-01-01')
    .select(['precipitation'])
    .sort('system:time_start');

// print(im_coll);
// Map.addLayer(im_coll.first());

// Filter on date (Jan 1st until April 1st)
// Sum the data (creating one image)
// Clip to northern_Ghana shape
var cumSum = im_coll
              .filterDate('2022-01-01','2022-04-01')
              .sum()
              .clip(northern_Ghana);

// Define threshold (in mm)
var thresholdValue = 50;

// Create mask, using threshold value
var mask = cumSum.lt(thresholdValue);

// Define colour palette and visualisation parameters 
var palette =  ['000096','0064ff','00b4ff','33db80','9beb4a',
                'ffeb00','ffb300','ff6400','eb1e00','af0000'];
var prec_vis = {min: 0.0, max: 100.0, palette: palette}; 

Map.addLayer(cumSum, prec_vis, 'Summed precipitation up to date [mm]', 1, 0.6 );
Map.addLayer(mask,{min: 0.0, max: 1.0} , 'Threshold Exceeded up to date [yes/no]', 1, 0.6 );
```

## 1.3.	Cumulative precipitation in a function
We will now apply this method in a **function** to determine the cumulative precipitation for each day of the year. We also write a function to determine whether the threshold is exceeded for each image in the image collection. This will help us to determine what day of year a certain threshold is crossed for a specific year. 

```js
// Function to determine the cumulative precipitation for each day (in selection of dates)
var calculateCumulativeSum = function(image){
  // for the current image, get the date
  var date = ee.Date(image.get('system:time_end'));
  // sum the rainfall for all images up to the current image
  var cumulativeSum = im_coll.filterDate('2022-01-01', date).sum();
  // clip the area and rename the band
  var cumulativeSum_clipped  = cumulativeSum.clip(northern_Ghana).rename('cumSum');
  // add the cumulative sum as a band to the image and store it as new_image
  var new_image = image.addBands(cumulativeSum_clipped);
  return new_image};

// Function to determine whether the threshold is exceeded for each image in the image collection
var calculateThresholdExceedence = function(image){
  var mask = image.lt(thresholdValue).rename('mask');
  // add the mask as a band to the image and store new_image
  var new_image = image.addBands(mask);
  return new_image};
```

Finally, we apply those functions and create a map of the day of exceedance in 2022 for each pixel.

```js
// calculate the cumulative precipitation for each day of the year
var prec_cumSum = im_coll.map(calculateCumulativeSum).select('cumSum');
// print(prec_cumSum)

// calculate if the threshold has been exceeded for each day of the year
var mask_threshold = prec_cumSum.map(calculateThresholdExceedence).select('mask');
// print(mask_threshold)

// determine the day of exceedence of the threshold at each pixel and add to the map
// this action takes some time
var doy_threshold = mask_threshold.sum();
Map.addLayer(doy_threshold, 
          {min:60,  max: 130, palette: ['blue', 'red']}, 
          'DOY Exceedence', 1, 1);
```

Your result should look something like this:

![image](https://gist.github.com/assets/5186265/b8e2c53b-0d01-43f1-ad10-599693d4bf59)

**Additional exercise: Redo this assignment with a threshold of 70 mm and analyse the difference with the 50 mm threshold in terms of rainfall onset.**

# 2	Day of Year (DoY) rainfall onset for multiple years

Now that we have been able to calculate the day of year of the rainfall onset for one year, we will do this for multiple years. We collect the result as a single image, but with bands for each year, that we can visualise separately. We will also export the image to our Google Drive, so that we can work with the result outside of the Google Earth Engine as well. 

## 2.1.	Define area of interest and select values

We start with defining the area of interest, which is still the White Volta region. Do you get how we define this area? Play around with it and make sure your area of interest is captured!
Then, we center the map and add the area of interest. 
We define the threshold value and start and end year of the analysis. 

```js
// Start with defining the area of interest (still the White Volta)
var aoi_whitevoltabasin = ee.Geometry.Polygon(
  [[[-3.2, 11.5],
    [-3.2, 8.5],
    [0.9, 8.5],
    [0.9, 11.5]]], null, false);
          
Map.centerObject(aoi_whitevoltabasin, 7.5);
Map.addLayer(aoi_whitevoltabasin);

// Our threshold in mm
var thresholdValue = 50;

// Definition of years that we like to process
// Do you know what years are included and not? 
// In this case both the start year and the end year are included in the analysis. 
var startYear = 1981; //did you check form when to when this data is available?
var endYear = 2023;
```

## 2.2.	Define a function to calculate DoY for multiple years

The power of earth engine is the capability to perform our computation on a large archive of data. 
Still you will have to make sure that the computation is happening on the server of google and that intermediate results are not returned to our screens. 
To make this happen, we define a function within a function. We send this to the server of google and only collect result to our screen.
Our function within a function follows these steps:

* 1\. Create an image collection with filtered images of a given year. 
* 2.1\. Compute the cumulative sum of precipitation. 
* 2.2\. Detect within the cumulative sum for each day of the year if the threshold has passed. 
* 2.3\. Return pixel-wise the minimum, which is the day of year (DOY) where the threshold was passed first.  

**Can you follow what each step does?**

```js
var processYear = function (year, image) {
  year = ee.Number(year);
  var startDate = ee.Date.fromYMD(year, 1, 1);
  var endDate = startDate.advance(1, 'year');

  var im_coll = ee.ImageCollection('UCSB-CHG/CHIRPS/DAILY')
    .filterBounds(aoi_whitevoltabasin)
    .filterDate(startDate, endDate)
    .select(['precipitation'])
    .sort('system:time_start');

  var combinedCollection = im_coll.iterate(function(image, acc) {
    // define our input images
    image = ee.Image(image);
    acc = ee.ImageCollection(acc);
    var last = ee.Image(acc.get('last'));
  
    // compute cumulative sum
    var cumSum = last.add(image).copyProperties(image, image.propertyNames());
    cumSum = ee.Image(cumSum).rename('cumsum_precip');
  
    // compute threshold DOY
    var mask = cumSum.gt(thresholdValue);
    var maskedImage = cumSum.updateMask(mask);
    var doy = ee.Date(image.get('system:time_start')).getRelative('day', 'year').add(1);
    var doyImage = maskedImage.where(mask, doy);
    doyImage = ee.Image(doyImage).rename('doy_threshold');
  
    return acc
      .merge(ee.ImageCollection([doyImage]))
      .set('last', cumSum);
  }, ee.ImageCollection([]).set('last', ee.Image(0).rename('precipitation')));
  
  return ee.ImageCollection(combinedCollection).min();
};
```

## 2.3	Create a collection of images

Using the function above we collect a single image for each year. Using our defined start and end year we are going to create a collection of images, one for each year.

```js
var imagesPerYear = ee.List.sequence(startYear, endYear).map(function (year) {
  var processedImage = processYear(year);
  return processedImage.rename(ee.String('doy_threshold_').cat(ee.String(ee.Number(year).toInt())))});
```

## 2.4 Store the images

The result is an image collection, but we like to store each year as a band within a single image.
For example, a true-colour image has 3 bands, one band for the colour red, one for the colour green and one for blue. We like to define an image where each band represent the result of a single year.

```js
// We get the first image in the list
var firstImage = ee.Image(imagesPerYear.get(0));

// We define a function to append bands to the initial image
var appendBands = function (image, initial) {
  initial = ee.Image(initial);
  image = ee.Image(image);
  return initial.addBands(image)};

// We then iterate over the image collection and append each image as band to the initial image
var combinedImage = ee.Image(imagesPerYear.slice(1).iterate(appendBands, firstImage));

// Check check, print the resulting combined image
print('Combined Image', combinedImage);
```

## 2.5.	Create the images and add them to the map

Caution: Running the task so far within this script is relatively quick, but visualising the results of each year as a single layer to the map within earth-engine is slow...

To avoid issues with this: do not zoom in or out once you run the rest of the script with the visualisation of the layers. Each time you change your zoom level, the computation restarts.

Once all layers are computed (no grey colours in the Layers menu), you can slide all opacity sliders to zero. Then start from the bottom and turn them one-by-one to full. Like this you can see the variation over the years. 

![image](https://gist.github.com/assets/5186265/d03d2d2c-11c1-4aed-b1e1-ddee1a61ad6a)

![image](https://gist.github.com/assets/5186265/dd5f9ce9-d8d5-4f50-a7ef-a1e18ddf0d09)

```js
// Get the band names of the image, this should contain all years
var bandNames = combinedImage.bandNames();

// Define the colour palette
var pal = ['#0077BB', '#33BBEE', '#009988', '#EE7733', '#CC3311', '#EE3377'];

// Display each band as a separate layer in our map viewer
bandNames.evaluate(function(names) {
  names.forEach(function(name) {
    var singleBandImage = combinedImage.select([name]);
    Map.addLayer(
      singleBandImage.clip(aoi_whitevoltabasin), 
      {min: 50, max: 200, palette: pal}, 
      name
    );
  });
});
```

## 2.6 Export results

To continue working with our result we export the result to our Google Drive.

```js
// First, we define the export parameters.
var exportParams = {
  image: combinedImage,
  description: 'exported_image_doy_threshold',
  scale: 5000,  // set the scale according to your requirements
  region: aoi_whitevoltabasin,  // define the region of interest
  fileFormat: 'GeoTIFF',  // specify the file format for the export
};

// Export the image to Google Drive. You will have to approve this run in the `Tasks` tab.
Export.image.toDrive(exportParams);
```

## 2.7. Histogram
Lastly, we will visualise the results in a histogram.

```js
// Get the histogram data
var histogram = ui.Chart.image.histogram(combinedImage, aoi_whitevoltabasin, 5000);

// Print the histogram to the console
print(histogram);
```

## 2.8. Bonus: visualisation of results in Vega-Altair

We also can process the exported image outside earth-engine with python and visualise the results using Altair (see https://altair-viz.github.io/).

To see this visualisation of the results as prepared by the trainers, go to the following url: https://vega.github.io/editor/#/gist/482f1e34cb848e7e0cc295836be8c39c/doy_thresholds_chart.json/view

See this colab notebook how the above chart was created: https://colab.research.google.com/drive/1rD7Q7BvSVPQImj24uQWN2jKKSkYwaypQ?usp=sharing

# 3. Analysis and visualisation of rainfall onset in GIS

## Type of GIS software**

In this manual we will guide you through the steps using QGIS. We encourage you to use this software, but you are free to use any other GIS software to perform these steps, like ArcGIS. The trainers usually work with QGIS, so they will be better able to assist you in this software. If you choose to work with ArcGIS of course, the trainers will assist you as good as they can. 

Why we are a fan of QGIS? It is open source, free for everyone and has very good manuals (including online forums etc to get answers to all of your questions). 
QGIS installation (if needed)

If you decide to work with QGIS, you can work with any version of QGIS. If you still need to install QGIS, we advice you to download the long-term release (stable version). The latest stable version of QGIS is 3.34. You can find it here: https://qgis.org/en/site/forusers/download.html# Make sure to click “Get QGIS 3.34”. 

![image](https://gist.github.com/assets/5186265/8ad4d0b4-af89-44e0-bc42-45a49daa2268)

## 3.1	Importing results in GIS

First we **download our exported GEE files** from our Google Drive. 
After placing the file in the desired folder (e.g. make a folder named “Training_HKV” with a sub-folder named “Data”), we **import it in QGIS**. Importing can be done by either:

- Drag and drop from your folder into QGIS

![image](https://gist.github.com/assets/5186265/a05c4c06-722e-42ea-bdf8-cd88fd0e9154)

- Import via Layer _– Add layer – Add raster layer_

![image](https://gist.github.com/assets/5186265/97d985aa-9ac6-49e4-b693-59c2bd6b466c)

![image](https://gist.github.com/assets/5186265/89372440-72ca-4413-97a7-75faf27f967a)

You will see that QGIS assigned RGB colours to bands, creating some kind of art on a white map.

## 3.2	Add a background

First of all, to check if we imported data that is in the correct location, we will **add a background map: OpenStreetMap**. You can add this map via the OpenLayersPlugin. If you don’t have this plugin yet, we will first have to install it via _Plugins – Manage and install plugins_. Search for the OpenLayersPlugin and install it. 

![image](https://gist.github.com/assets/5186265/928f8a3c-e789-4481-bbda-3b65cdaa38d1)

If it is installed, you can add the OpenStreetMap layer to your project via _Web – OpenLayersPlugin – OpenStreetMap_. The layer will appear in the layers view on the left. You can drag and drop it below the other layer, so they are in the right order on the map: the **DoY should be projected over the OpenStreetMap**.

![image](https://gist.github.com/assets/5186265/0b9d7a8b-51b7-406e-82dd-f143a4fa3b09)

By scrolling in the map view you can zoom in and out. **You can see that the layer is in the right position (if all went well)**. 

Now that we added the layers, it is time to **save the project** so we won’t loose our work. Go to _Project – Save as_ and save your project, for example in the “Training_HKV” folder naming it “QGIS_master”.

We will now go a bit faster in the manual. If you have any difficulties following the steps, please ask your trainers!

## 3.3	Visualise the DoY layer

As you have seen the DoY layer that we added has been assigned RGB (red, green, blue) colours to band 1, 2 and 3. This is a common way of visualizing .tif files with multiple bands, however, this is not the way we want to visualise our data!

Do you remember what each band represents? 

**We want to visualise just one band at a time**! We will do that now by _double clicking on the DoY layer or right clicking and going to properties_. You will now be in the Layer properties menu. Go to the tab (on the left) called _Symbology_. Here we can change the rendering type of the various bands (how are they interacting and how are they displayed). 

Change the _Render type_ to _Singleband pseudocolor_. Do you get what that means? And what would the other render types mean? 

We will now have to choose a band to visualise and Min/Max value settings. We will start off with a random band. Click through the bands. Do you see what happens to the min/max? It might change depending on the band you click and depending on the settings. We want to choose a min/max that is representative for all bands so we can compare the different bands. To check what would be appropriate min/max values, we will analyse this plot of our data (analysed in the bonus question): 

![image](https://gist.github.com/assets/5186265/69b085f8-5d13-49dc-934e-b076faba8e46)

Choose a min/max that are appropriate for all/most bands and use this in the layer visualisation. Make sure to make the min/max settings “user defined” (expand this menu). Otherwise the min/max might change per band. 

Now choose a color ramp of your choice. 

You settings may now look like this (but make your own choices!): 

![image](https://gist.github.com/assets/5186265/753de785-4957-4248-9a88-0f31cdddf573)

## 3.4	Explore the information about the layer

Now, explore what other information there is about the DoY layer in the tabs of the Layer Properties (open the layer properties by double clicking on the layer in the layers viewer). 

-	What is the extent of the dataset?
-	What is the pixel size of the raster?
-	Can you make the layer (partially) transparent? 
-	Where is the dataset stored?
-	What is the CRS (coordinate reference system) the file is in?
-	Can you find the characteristics (max, min, median, etc) of the various bands? 

## 3.5	Analyse the data in the DoY raster layer

Now that we have loaded and visualized the DoY layer, and we have also explored the general properties of the layer, we will analyze the data in more detail. 

First we do some raster calculations: we analyse the minimum, average and maximum of all DoY layers from 2015 – 2023 (these are the years we will generate DoY flood maps for in future exercises) and/or for all DoY layers from 1981-2023 (start of the data). 

We do this with the raster calculator. Here we give an example for calculating the maximum between 2015 and 2023 in the raster calculator: 

```
MAX( MAX( MAX( MAX( MAX( MAX( MAX( MAX("DoY_2000_2023@16","DoY_2000_2023@17"),"DoY_2000_2023@18"),"DoY_2000_2023@19"),"DoY_2000_2023@20"),"DoY_2000_2023@21"),"DoY_2000_2023@22"),"DoY_2000_2023@23"),"DoY_2000_2023@24")
```

Do you get what we do here? Try to do it yourself for your project and/or timespan. 
Also do it for the minimum and the average. 

One of the ways to analyse multiple raster layers at once is with the value tool. You have to install this plugin via the plugins menu and activate it.
What do you see? What is worth mentioning? Discuss this.

## 3.6	Sample raster values in point layer

Now that we have generated some raster information, we will make two vector layers: one with points and one with areas. 

We can make a vector layer with points by: 
_Layer – Create layer – New geopackage layer_. 
Define the location where to store the layer (in de map you made), select that you want to make a layer with point geometries and make the layer by clicking ok. Now that we have made this layer, we can add points by selecting the layer on the layers panel, and then select _Toggle editing_ and _Add point feature_. Now you can click on the map where you want to add point features. We will do this along the White Volta at +/- 8 to 10 locations that we will analyse. When you are finished, deselect _Toggle editing_ and save your changes of the layer. Also save the project.

![image](https://gist.github.com/assets/5186265/199a13bc-fffb-4c70-8aba-c1e31de3b97b)

Now we will combine the raster and point information by sampling raster values at the points. We do this by going to the processing toolbox  . A new window will appear. Have a look what is possible. 

Now search for “sample” and look for the Sample raster values function. We now sample the values of the raster bands of the original DoY layer for the point that we just created. Make sure to store your sampled new layer as gpkg (GeoPackage). Give it a name so you understand what you did. 

![image](https://gist.github.com/assets/5186265/63939edb-5d89-4f26-a355-62011db7c13e)

Analyse your new layer. What do you see and what data do we have now? For example in the attribute table? 

## 3.7	Analyse the results in Excel

Now that you have analyzed this in QGIS, it might be easy to get an overview of these results. We will analyse this in Excel. We can do this by opening the attribute table of the new point layer with sampled values, then selecting all cells and copying their values (ctrl + c). Paste this in a new excel file. 

![image](https://gist.github.com/assets/5186265/d167ddfa-8d79-4059-ab92-c6a2a2eb7654)

Now that we have these values in Excel, play around analysing the data. What can you learn and how? For example, play around with finding maximum, minimum, average, but also visualizing, plotting in graphs etc. Have fun and be critical of what you see!

## 3.8	Composing maps

Compose a map containing the DoY of the rainy season onset of (at least) one year. 

A good map should at least contain:

-	Your chosen map view
-	A north arrow
-	A scale bar
-	A title
-	A legend

How you want to present this, with what layers, what names, etc. is completely up to you! 

We can help you if you have any questions or need any suggestions for your map. 

Creating a map view in QGIS: Go to the _Layout manager – Create an empty layout – Name it something like “DoY map”_. We can now compose a map. The buttons on the left of the map composer will be of great help to add all elements to your map. In the windows on the right you can adjust all the settings of the elements in your map. 

![image](https://gist.github.com/assets/5186265/048ea4f7-3a96-41c2-932b-4d2833561100)

# 4.	Day of Year (DoY) flood onset

Back to a new chapter. In this chapter we will calculate the onset of a flood. In this chapter we will do this for a single year. After we understand the principles including the pros and cons we will do it in the next chapter for multiple years. Ready? Lets go!

Do you still remember? 

•	What is Day of Year? 
•	Answer the following: Today is …. day in the year
•	Which months is the white volta basin sensitive to flooding?

You can use:

-	https://www.easysurf.cc/wdate5.htm

## 4.1	Define area and visualise region

Load the following envelope in a new script

```js
var aoi_whitevoltabasin = ee.Geometry.Polygon([[
  [-1.45, 11.02],
  [-1.45, 8.55],
  [-0.32, 8.55],
  [-0.32, 11.02]
]]);

Map.addLayer(aoi_whitevoltabasin)
Map.centerObject(aoi_whitevoltabasin, 7)
```

The result will look like this:

![image](https://gist.github.com/assets/5186265/549ad178-16af-4233-8be1-a761ae3d23a2)

## 4.2	Define ImageCollection

What we like to do for this declared envelope is to find out the Day of Year for each pixel in our area of interest when it equals water. We first start with developing our ImageCollection in which we are going to do this.
As said, we will do this for a single year first:

```js
// define our year of interest
var year = 2021;
var startDate = ee.Date.fromYMD(year, 1, 1);
var endDate = startDate.advance(1, 'year');

// Filter the Sentinel-1 ground range detected for the specified time range and region.
var im_coll = ee.ImageCollection('COPERNICUS/S1_GRD')
  .filterBounds(aoi_whitevoltabasin)
  .filterDate(startDate, endDate)
  .filter(ee.Filter.calendarRange(210, 274, 'day_of_year'))  // 07-25 : 10-01 
  .filter(ee.Filter.listContains('transmitterReceiverPolarisation', 'VV'))  
  .sort('system:time_start');

print(im_coll)
```

The result will look like this:

![image](https://gist.github.com/assets/5186265/baa56a61-6d69-424e-a033-02c1a623e37a)

So what do we do here?
Can you explain in your own words what the following means:
-	ee.ImageCollection()
-	filterBounds()
-	filterDate()
-	filter(ee.Filter.calendarRange())
-	filter(ee.Filter.listContains())

## 4.3	Visualise a SAR Image

We have learned that SAR data is merely just reflecting the state of the terrain. Bare soil is very smooth just like water, so in SAR imagery this will be reflected similar.

We will have a look at the junction where the Kulpawn streams into the White Volta. Zoom to here and use the following code.

Lets visualise it using a grayscale

```js
// ioi, Image of Interest. Using Index values we can select each image
// in our image collection to inspect it.
var index_no = 3;  // 3, 7, 11, 15, 19, 23, 27, 31, 35, 39
var ioi = ee.Image(im_coll.toList(im_coll.size()).get(index_no)).select('VV');

// Get acquisition date
var acquisitionDate = ee.Date(ioi.get('system:time_start'));

// Combine acquisition date and day of year
var dayOfYear = ee.String(acquisitionDate.getRelative('day', 'year'));
var label = ee.String('Acquisition Date: ')
  .cat(acquisitionDate.format('YYYY-MM-dd'))
  .cat(' Day of Year: ')
  .cat(dayOfYear);

print(label)

var vizParams = {
  min: -25,
  max: -5, 
  palette: ['black', 'white'] // Grayscale color palette
};

Map.addLayer(ioi.clip(aoi_whitevoltabasin), vizParams, 'Sentinel-1 VV')
```

This will add a layer for the following acquisition date

```
Acquisition Date: 2021-08-03 Day of Year: 214
```

Question, using this code, :
-	When is the flood onset starting? Which day of year is this?
-	At which day of year is the flood at its maximum?
-	At which day of year is the flood finished and is the white volta back to its normal

Nice looking SAR picture:

![image](https://gist.github.com/assets/5186265/d1e07d39-3d03-4f1d-b028-cc7c19daffbd)

Use the point inspector and check for a fews pixels what is the value of what you think represents water and what you think represents crops.

![image](https://gist.github.com/assets/5186265/68ff672b-9887-488d-876c-bba8ff197b6f)

## 4.4	Focal Statistics and Morphological Operations Theory

SAR data is a bit noisy. So we add some filters to smooth the image. We will use focal statistics and morphological operations for this. In particular it will be a focal median smoothing and apply a dilation as morphological operation.

![image](https://gist.github.com/assets/5186265/97edb4d1-6716-4b22-ac1a-1b70157755fa)

What is the median of the following range?  0,1,2,2,3,3,4,4,5

Morphological operations:
![image](https://gist.github.com/assets/5186265/edb2a0fd-2f37-416f-aa99-f551119f807a)

In this case we will use a dilation as morphological operation. 
See also https://link.springer.com/chapter/10.1007/978-3-031-26588-4_10 

![image](https://gist.github.com/assets/5186265/0804220d-7d53-4e80-ae72-0714dbd92249)

![image](https://gist.github.com/assets/5186265/2f06e4a4-d470-4ba3-85b9-47b21d2ffef1)

## 4.5	Focal Statistics and Morphological Operations Code

OK, so let’s apply it to in code for our use case

```js
// Define a square, uniform kernel (smooting).
var uniformKernel = ee.Kernel.square({
  radius: 3,
  units: 'meters',
});

var vv_smoothed = ioi.focal_median(100,'circle','meters').rename('VV_Filtered') // default is 100

// identify all pixels below threshold and set them equal to 1. All other pixels set to 0
var watermask = vv_smoothed.lt(-13).rename('Water')  //default is 13

// apply a dilation as morphological operation
watermask = watermask.reduceNeighborhood({
  reducer: ee.Reducer.max(),
  kernel: uniformKernel
}).rename('Water');    

var maskedImage = watermask.updateMask(watermask).float()  
// Define visualization parameters
var vizParamsWater = {palette: ['blue']};
Map.addLayer(maskedImage.clip(aoi_whitevoltabasin), vizParamsWater, 'Water Smoothed')
```

The result will look like this

![image](https://gist.github.com/assets/5186265/57542d2b-da77-477a-81d1-ea68b4a7fe45)

## 4.6	Flood onset Day of Year

So to determine in a single year for each pixel the first day of year when water is detected. We do a nice trick. We set the day of year to each pixel where water was detected in our maskedImage. Like this:

```js
// compute the day of year of the image, burn into the water mask
var doy = ee.Date(ioi.get('system:time_start')).getRelative('day', 'year').add(1);
var doyImage = maskedImage.where(watermask, doy);

// add the doy_water as new band to the retured image
doyImage = ee.Image(doyImage).rename('doy_water');

// Add a palette and clip the image by country
var vizDoY = {min:210, max:275, palette:['c10000','d742f4','001556','b7d2f7']};
Map.addLayer(doyImage.clip(aoi_whitevoltabasin), vizDoY, 'DoY Water Smoothed')
```

From the layer you will not see a big difference. Just another layer with a similar color as the smoothed water layer. But now inspect a few points and you will see something like this:

![image](https://gist.github.com/assets/5186265/30d75b18-dc30-4142-a6b8-3060130603b8)

We have burned the day of year into each pixel that is detected as water.

Great

## 4.7	Flood onset Day of Year by Function

We have done the above more or less line by line, but we also can group this smoothing into a separate function that is easier to call. Lets do it:

```js
// Now we will define the above section as a function
// Function to process a single image
var processImage = function(image) {
  // select the VV polarization band
  // apply a focal median filter, to reduce speckle
  var vv = image.select('VV');
  var vv_smoothed = vv.focal_median(100,'circle','meters').rename('VV_Filtered'); // default is 100
  
  // identify all pixels below threshold and set them equal to 1. All other pixels set to 0
  var watermask = vv_smoothed.lt(-13).rename('Water');  //default is 13

  // apply a dilation as morphological operation
  watermask = watermask.reduceNeighborhood({
    reducer: ee.Reducer.max(),
    kernel: uniformKernel
  }).rename('Water');    
  
  var maskedImage = watermask.updateMask(watermask).float();  
  
  // compute the day of year of the image, burn into the water mask
  var doy = ee.Date(image.get('system:time_start')).getRelative('day', 'year').add(1);
  var doyImage = maskedImage.where(watermask, doy);
  
  // add the doy_water as new band to the retured image
  doyImage = ee.Image(doyImage).rename('doy_water');  
  return image.addBands(doyImage);
};

var ioi_doy_water = processImage(ioi);
Map.addLayer(ioi_doy_water.select('doy_water').clip(aoi_whitevoltabasin), vizDoY, 'DoY Water Smoothed by Function')
```

As you can see using the inspector the result is the same in the inspector:

![image](https://gist.github.com/assets/5186265/5d10dea3-a08c-4af7-a1c6-c01e7a2e9fcc)

But we have reduced the process for a single image to:

```js
var ioi_doy_water = processImage(ioi);
```

## 4.8	Apply Function to ImageCollection

We now apply this function to our whole image collection that we have defined in section 4.2
As such:

```js
var combinedCollection = im_coll.map(processImage);

print(combinedCollection)
```

Awesome, awesome, awesome. All this difficult matter reduced to a single line of code!
Inspect that we have added a band doy_water to each image in the imageCollection:

![image](https://gist.github.com/assets/5186265/60c197e6-2403-4e2b-adbf-98cbb00d7522)

## 4.9	Reduce ImageCollection to image with first day of year with water

Almost there! 
Last bit is to get the pixel-wise minimum. So we can get the first day of the water detected.
Lets do it!

```js
// get pixel-wise .min() value to get first day of year with water detected
var yearImage = ee.ImageCollection(combinedCollection.select('doy_water')).min()

Map.addLayer(yearImage.clip(aoi_whitevoltabasin), vizDoY, "Flood onset single year")
```

The result will look like this:

![image](https://gist.github.com/assets/5186265/79e43de1-7f77-4357-938e-5fb27f8d34b4)

Beautiful!
Can you interpret the map in your own words:

```
…
```

If you like, you can make a histogram to get a better understanding:

```js
// Get the histogram data
var histogram = ui.Chart.image.histogram(yearImage, aoi_whitevoltabasin, 100);

// Print the histogram to the console
print(histogram);
```

![image](https://gist.github.com/assets/5186265/7c4eafbe-8ba3-46f3-b1e1-dfbc9a4437b8)

What do you think now? Does this change your interpretation?

## 4.10	Beautified code

The following is a clean up of all the code above. You can open this in a new script:

```js
// define an envelope for our area of interest
var aoi_whitevoltabasin = ee.Geometry.Polygon([
    [
        [-1.5, 11.1],
        [-1.5, 8.5],
        [-0.2, 8.5],
        [-0.2, 11.1]
    ]
]);

// define our year of interest
var year = 2021;
var startDate = ee.Date.fromYMD(year, 1, 1);
var endDate = startDate.advance(1, 'year');

// Filter the Sentinel-1 ground range detected for the specified time range and region.
var im_coll = ee.ImageCollection('COPERNICUS/S1_GRD')
  .filterBounds(aoi_whitevoltabasin)
  .filterDate(startDate, endDate)
  .filter(ee.Filter.calendarRange(210, 274, 'day_of_year'))  // 07-25 : 10-01 
  .filter(ee.Filter.listContains('transmitterReceiverPolarisation', 'VV'))  
  .sort('system:time_start');

// Define a square, uniform kernel (smooting).
var uniformKernel = ee.Kernel.square({
  radius: 3,
  units: 'meters',
});

// Function to process a single image
var processImage = function(image) {
  // select the VV polarization band
  // apply a focal median filter, to reduce speckle
  var vv = image.select('VV');
  var vv_smoothed = vv.focal_median(100,'circle','meters').rename('VV_Filtered'); // default is 100
  
  // identify all pixels below threshold and set them equal to 1. All other pixels set to 0
  var watermask = vv_smoothed.lt(-13).rename('Water');  //default is 13

  // apply a dilation as morphological operation
  watermask = watermask.reduceNeighborhood({
    reducer: ee.Reducer.max(),
    kernel: uniformKernel
  }).rename('Water');    
  
  var maskedImage = watermask.updateMask(watermask).float();  
  
  // compute the day of year of the image, burn into the water mask
  var doy = ee.Date(image.get('system:time_start')).getRelative('day', 'year').add(1);
  var doyImage = maskedImage.where(watermask, doy);
  
  // add the doy_water as new band to the retured image
  doyImage = ee.Image(doyImage).rename('doy_water');  
  return image.addBands(doyImage);
};

// apply function to our image collection
var combinedCollection = im_coll.map(processImage);

// get pixel-wise .min() value to get first day of year with water detected
var yearImage = ee.ImageCollection(combinedCollection.select('doy_water')).min()

// Add a palette and clip the image by country
var vizDoY = {min:210, max:275, palette:['c10000','d742f4','001556','b7d2f7']};
Map.addLayer(yearImage.clip(aoi_whitevoltabasin), vizDoY, "Flood onset single year")
```

There are some defaults in the code above, like 100 meter for the focal median and -13 for the water threshold and 210:274 for the calender range. 

-	Can you adjust these values to see what is the impact? 
-	Just play with the numbers. 
-	Eg change focal median from -13 to -15 
-	And change calendar range from 210:274 to 150:274.

# 5.	Do it for multiple years

We will adapt our code of 4.10 so we can calculate this for multiple years. That means we add another function which we call `processYear`. In this function we will call our created function `processImage`. 

## 5.1	Define function to process a single year

The code looks as follow:

```js
var aoi_whitevoltabasin = ee.Geometry.Polygon([[
  [-1.45, 11.02],
  [-1.45, 8.55],
  [-0.32, 8.55],
  [-0.32, 11.02]
]]);

// Define a square, uniform kernel (smooting).
var uniformKernel = ee.Kernel.square({
  radius: 3,
  units: 'meters',
});

// Function to process a single image
var processImage = function(image) {
  // select the VV polarization band
  // apply a focal median filter, to reduce speckle
  var vv = image.select('VV');
  var vv_smoothed = vv.focal_median(100,'circle','meters').rename('VV_Filtered'); // default is 100
  
  // identify all pixels below threshold and set them equal to 1. All other pixels set to 0
  var watermask = vv_smoothed.lt(-13).rename('Water');  //default is 13

  // apply a dilation as morphological operation
  watermask = watermask.reduceNeighborhood({
    reducer: ee.Reducer.max(),
    kernel: uniformKernel
  }).rename('Water');    
  
  var maskedImage = watermask.updateMask(watermask).float();  
  
  // compute the day of year of the image, burn into the water mask
  var doy = ee.Date(image.get('system:time_start')).getRelative('day', 'year').add(1);
  var doyImage = maskedImage.where(watermask, doy);
  
  // add the doy_water as new band to the retured image
  doyImage = ee.Image(doyImage).rename('doy_water');  
  return image.addBands(doyImage);
};

// Function to process a single year
var processYear = function(year) {
  var startDate = ee.Date.fromYMD(year, 1, 1);
  var endDate = startDate.advance(1, 'year');
  
  // Filter the Sentinel-1 ground range detected for the specified time range and region.
  var im_coll = ee.ImageCollection('COPERNICUS/S1_GRD')
    .filterBounds(aoi_whitevoltabasin)
    .filterDate(startDate, endDate)
    .filter(ee.Filter.calendarRange(210, 274, 'day_of_year'))  // 07-25 : 10-01 
    .filter(ee.Filter.listContains('transmitterReceiverPolarisation', 'VV'))  
    .sort('system:time_start');  
      
  // apply function to our image collection
  var combinedCollection = im_coll.map(processImage);
  
  // get pixel-wise .min() value to get first day of year with water detected
  return ee.ImageCollection(combinedCollection.select('doy_water')).min()
};
    
// define our year of interest
var year = 2021;
var processedYear = processYear(year);
    
// Add a palette and clip the image by country
var vizDoY = {min:210, max:275, palette:['c10000','d742f4','001556','b7d2f7']};
Map.addLayer(processedYear.clip(aoi_whitevoltabasin), vizDoY, "Flood onset single year")
```

The result should be the same as the code in section 4.10.
Observe that we have reduced your code to process a single year to only this:

(no need to copy this)

```js
// define our year of interest
var year = 2021;
var processedYear = processYear(year);
```

Quite awesome! 

Now we can compute it for another year. 2022? 2017? 

But be careful with the defaults!

## 5.2.	Apply function to multiple years

We define a list of years over we are going to iterate. For each year we will apply our `processYear` function. We will rename our image so it include the year that we are processing. 

How many years are we going to process? Check when the sentinel 1 product starts and fill in your value for `startYear`.

```js
// next is the definition of years that we like to process. 
var startYear = <FILL IN YOUR VALUE>; 
var endYear = 2023;

// using the function above we can collect a single image for each year in a list
var imagesYears = ee.List.sequence(startYear, endYear).map(function (year) {
  var processedYear = processYear(year);
  return processedYear.rename(ee.String('doy_water_').cat(ee.String(ee.Number(year).toInt())));
});

print(imagesYears)
```

You will see that this returns a list with images:

![image](https://gist.github.com/assets/5186265/675381bc-b2b4-4850-ad32-5540b2e926ec)

## 5.3.	From list of images to bands in single image

The result is a list of images. but we like to store the image of each year as a band within a single image. To do this we
1.	first define a single image and
2.	iterate over our remaining list of images and 
3.	add this to our defined initial image

```js
// we get the first image in the list
var firstImage = ee.Image(imagesYears.get(0));

// we define a function to append bands to the initial image
var appendBands = function(image, initial) {
  initial = ee.Image(initial);
  image = ee.Image(image);
  return initial.addBands(image);
};

// we then iterate over the list and append each image as band to the initial image
var combinedImage = ee.Image(imagesYears.slice(1).iterate(appendBands, firstImage));

// check check, print the resulting combined image
print('Combined Image', combinedImage);
```

So from a list of images we went to a single image with a band for each year:

![image](https://gist.github.com/assets/5186265/cb8f1a79-a88a-45d3-b636-2dee58788447)

## 5.4.	Visualize each band as a single layer

We define a little code snippet that iterate over all bands and add it as a single layer to the map. Be careful, don’t pan around when the function is running. First zoom to your area of interest and then run the code

```js
// Get the band names of the image, this shoud contain all years
var bandNames = combinedImage.bandNames();

// display each band as a separate layer in our map viewer
bandNames.evaluate(function(names) {
    names.forEach(function(name) {
        var singleBandImage = combinedImage.select([name]);
        Map.addLayer(
            singleBandImage.clip(aoi_whitevoltabasin),
            vizDoY,
            name
        )
    })
});
```

![image](https://gist.github.com/assets/5186265/7c5c0e11-3c1f-4900-ae31-1b066956cc0d)

Instead of turning on and off layers. It is better to reduce opacity of all years that you are not interested in and only increase the opacity of the year of interest.

Pretty nice! You did this!

## 5.5.	Export the image as an asset

OK, lets export the image so we can open it in our favorite GIS software!

```js
// to continue working with our result we export the result to our Google Drive.
// therefor we define the export parameters
var exportParams = {
  image: combinedImage,
  description: 'exported_image_doy_water_s1',
  scale: 20,  // set the scale according to your requirements
  region: aoi_whitevoltabasin,  // define the region of interest
  fileFormat: 'GeoTIFF',  // specify the file format for the export
};

// Export the image to Google Drive. You will have to approve this run in the `Tasks` tab.
Export.image.toDrive(exportParams);
```

We have defined a scale of 20 meters. 
-	Do you think this is a fine resolution? 
-	Can you remember what resolution we set for the precipitation data? 
-	Do you think the difference is appropriate? Why?

Run the code and run the task within the tasks tab

![image](https://gist.github.com/assets/5186265/5dece5f7-0c49-434d-94d4-097cf3fa1e20)

You've manifested your inner coding guru! 
From now on you will be guiding the way for the rest with your skills and expertise!

## 5.6.	Everything of chapter 4 & 5 in once

```js
var aoi_whitevoltabasin = ee.Geometry.Polygon([[
  [-1.45, 11.02],
  [-1.45, 8.55],
  [-0.32, 8.55],
  [-0.32, 11.02]
]]);       

// Define a square, uniform kernel (smooting).
var uniformKernel = ee.Kernel.square({
  radius: 3,
  units: 'meters',
});

// Function to process a single image
var processImage = function(image) {
  // select the VV polarization band
  // apply a focal median filter, to reduce speckle
  var vv = image.select('VV');
  var vv_smoothed = vv.focal_median(100,'circle','meters').rename('VV_Filtered'); 
  
  // identify all pixels below threshold and set them equal to 1. All other pixels set to 0
  var watermask = vv_smoothed.lt(-13).rename('Water');  //default is 13

  // apply a dilation as morphological operation
  watermask = watermask.reduceNeighborhood({
    reducer: ee.Reducer.max(),
    kernel: uniformKernel
  }).rename('Water');    
  
  var maskedImage = watermask.updateMask(watermask).float();  
  
  // compute the day of year of the image, burn into the water mask
  var doy = ee.Date(image.get('system:time_start')).getRelative('day', 'year').add(1);
  var doyImage = maskedImage.where(watermask, doy);
  
  // add the doy_water as new band to the retured image
  doyImage = ee.Image(doyImage).rename('doy_water');  
  return image.addBands(doyImage);
};

// Function to process a single year
var processYear = function(year) {
  var startDate = ee.Date.fromYMD(year, 1, 1);
  var endDate = startDate.advance(1, 'year');
  
  // Filter the Sentinel-1 ground range detected for the specified time range and region.
  var im_coll = ee.ImageCollection('COPERNICUS/S1_GRD')
    .filterBounds(aoi_whitevoltabasin)
    .filterDate(startDate, endDate)
    .filter(ee.Filter.calendarRange(210, 274, 'day_of_year'))  // 07-25 : 10-01 
    .filter(ee.Filter.listContains('transmitterReceiverPolarisation', 'VV'))  
    .sort('system:time_start');  
      
  // apply function to our image collection
  var combinedCollection = im_coll.map(processImage);
  
  // get pixel-wise .min() value to get first day of year with water detected
  return ee.ImageCollection(combinedCollection.select('doy_water')).min()
};

// next is the defintion of years that we like to process. 
var startYear = 2015;
var endYear = 2023;

// using the function above we can collect for each year a single image.
var imagesYears = ee.List.sequence(startYear, endYear).map(function (year) {
  var processedYear = processYear(year);
  return processedYear.rename(ee.String('doy_water_').cat(ee.String(ee.Number(year).toInt())));
});

// we get the first image in the list
var firstImage = ee.Image(imagesYears.get(0));

// we define a function to append bands to the initial image
var appendBands = function(image, initial) {
  initial = ee.Image(initial);
  image = ee.Image(image);
  return initial.addBands(image);
};

// we then iterate over the list and append each image as band to the initial image
var combinedImage = ee.Image(imagesYears.slice(1).iterate(appendBands, firstImage));

// to continue working with our result we export the result to our Google Drive.
// therefor we define the export parameters
var exportParams = {
  image: combinedImage,
  description: 'exported_image_doy_water_s1',
  scale: 20,  // our scale of interst
  region: aoi_whitevoltabasin,  // define the region of interest
  fileFormat: 'GeoTIFF',  // specify the file format for the export
};

// Export the image to Google Drive. You will have to approve this run in the `Tasks` tab.
Export.image.toDrive(exportParams);
```
