---
layout: post
title: Mapping Out Dipterocarp Forests with Remote Sensing
tags: GIS
---
## Cloud Masking of Sentinel-2 Data
First, we need clean data to work with. If we were using only the Normalised Difference Vegetation Index (NDVI), then using Copernicus' Sentinel-2 Quarterly Global Mosaics would be of no issue. But, as we are also creating a Normalised Difference Moisture Index (NDMI), which requires Band 11 (SWIR1), we can't use the Quarterly Global Mosaics.

### Loading Sentinel-2 Image Collection
This means that we need to create our own cloudless images to work with. To begin, we need to load the Sentinel-2 image collection into our Google Earth Engine file.

<code>
// Harmonized Sentinel-2 Level 2A collection.
var s2 = ee.ImageCollection('COPERNICUS/S2_SR_HARMONIZED');
</code>

We also need to choose our aoi (Area of Interest).

<code>
var aoi = ee.FeatureCollection("FAO/GAUL/2015/level0")
            .filter(ee.Filter.inList('ADM0_NAME', 
                    [
                      'Singapore',
                      'Malaysia',
                      'Brunei Darussalam'
                      ]));
</code>

### Cloud Masking with Sentinel-2 Imagery
To do some cloud masking, we will utilise the cloud scoring image collection, which is similar to the f mask.

<code>
// Cloud Score+ image collection. Note Cloud Score+ is produced from Sentinel-2
// Level 1C data and can be applied to either L1C or L2A collections.
var csPlus = ee.ImageCollection('GOOGLE/CLOUD_SCORE_PLUS/V1/S2_HARMONIZED');

// Use 'cs' or 'cs_cdf', depending on your use case; see docs for guidance.
var QA_BAND = 'cs_cdf';

// The threshold for masking; values between 0.50 and 0.65 generally work well.
// Higher values will remove thin clouds, haze & cirrus shadows.
var CLEAR_THRESHOLD = 0.70;

// Make a clear median composite.
var composite = s2
    .filterBounds(aoi)
    .filterDate('2025-01-01', '2025-12-01')
    .linkCollection(csPlus, [QA_BAND])
    .map(function(img) {
      return img.updateMask(img.select(QA_BAND).gte(CLEAR_THRESHOLD));
    })
    .median();
</code>

### Choosing The Required Sentinel-2 Bands
we will now choose Bands 2, 3, 4, 8, 11, and 12.
These are our Blue, Green, Red, Near Infrared, Short Wavelength Infrared 1, and Short Wavelength Infrared 2 bands.

<code>
// Pull the bands that you require.
var requiredBands = ['B2', 'B3', 'B4', 'B8','B11','B12'];
composite = composite.select(requiredBands);
</code>

## Creating the NDVI
Now, we will create a colour palette so we can actually visualise the NDVI and NDMI. This will be a very simple palette, with white representing 0, and black representing 1. You can change the colours to whatever you feel is visually appealing or easy to understand. Sometimes, I replace black ['000000'] with ['398f53'] as the dark green can easily represent forest cover.

<code>
// Make a single grey palette: two hex strings, white and black.
var palette = ['FFFFFF', '000000'];
</code>

The creation of the NDVI is easy, as Google Earth Engine has added a normalizedDifference function. It is used by typing your imageName.normalizedDifference(['band1','band2']). The NDVI specifically finds out the amount of chlorophyll present in a single pixel of the Sentinel-2 image. The more chlorophyll present, the closer the output would be to 1. If there is zero chlorophyll present, the output would be 0.

<code>
// Creation of NDVI via normalizedDifference(A, B) to compute (A - B) / (A + B)
var ndvi = composite.normalizedDifference(['B8', 'B4']);
</code>

To make this NDVI meaningful, we will add a threshold to how much chlorophyll would be required for that pixel to be deemed a Dipterocarp Forest (and usually a Mature Secondary Forest, as they are similar in nature). This threshold (0.739016) has been selected after I did some testing with the Central Catchment Nature Reserve, a forest I understand well enough.

<code>
var ndvi_forest = ndvi.expression(  // Use a threshold for forest remote sensing
'ndvi > 0.739016', {
  'ndvi' : ndvi
});
</code>

## Creating the NDMI
We will do the same, but now we are creating an NDMI instead. The NDMI finds out the amount of moisture content in plants. This is another factor that differentiates the Dipterocarp Forest from grasslands, as grassland phenology is usually drier than the very moist Dipterocarp Forest trees.

<code>
// Creation of NDMI via normalizedDifference(A, B) to compute (A - B) / (A + B)
var ndmi = composite.normalizedDifference(['B8','B11']);
var ndmi_forest = ndmi.expression(  // Use a threshold for forest remote sensing
  'ndmi > 0.309041', {
    'ndmi' : ndmi
});
</code>

## Merging the NDVI and NDMI
After we have created the NDVI and NDMI, we will now clip these two rasters into a singular image. The code below basically takes the two rasters and compares them. If a single pixel of nature fulfills both the chlorophyll and leaf moisture thresholds, it will be deemed Dipterocarp Forest. If that single pixel only fulfills one or zero thresholds, the pixel will be deemed as NOT Dipterocarp Forest.

<code>
// Create mean of NDMI and NDVI forest sensing
var poopoo = ee.ImageCollection([ndvi_forest, ndmi_forest]).mosaic();
var forest_detection = poopoo.expression(
  '((ndmi_forest + ndvi_forest) / 2) > 0.6', {
    'ndmi_forest' : ndmi_forest,
    'ndvi_forest' : ndvi_forest
});
</code>

## Visualising The Rasters
We can add lines of code that visualises our raster layers by adding them onto the map. One of the lines of visualisation code are commented out with a double slash '//'. You can use this if you wish to preserve the line of code, but don't want Google Earth Engine to execute it for now.

<code>
Map.centerObject(aoi, 6);
Map.addLayer(ndmi, {min: 0, max: 1, palette: palette}, 'NDMI');
Map.addLayer(ndvi, {min: 0, max: 1, palette: palette}, 'NDVI');
Map.addLayer(ndmi_forest, {min: 0, max: 1, palette: palette}, 'NDMI forest');
// Map.addLayer(ndvi_forest, {min: 0, max: 1, palette: palette}, 'NDVI forest');
Map.addLayer(forest_detection, {min: 0, max: 1, palette: palette}, 'Forest Detection');
</code>