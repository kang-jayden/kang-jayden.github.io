---
layout: post
title: Utilising Google's AlphaEarth in Classification of Land Use
categories: GIS
tags: GIS AlphaEarth Remote-Sensing Sentinel-2
---

## Some Context
### What is AlphaEarth?

Google Earth Engine and Google DeepMind came together and created a new dataset known as the Satellite Embedding Dataset. There are 64 embeddings created by running every single LandSat and Sentinel-2 image through a machine learning algorithm, which it then tagged every pixel to 64 "dimensions" or embeddings. Examples of these dimensions could be change in time (temporal), temperature, and surface reflectance of the staple few electromagnetic bands like SWIR and NIR. Every pixel found in the Satellite Embedding Dataset have been pansharpened to the moderately high resolution of 10m. 

### What is Land-Use Classification?

Land-Use Classification is the category of activity occuring on a certain land parcel. It could be slated for educational use, commercial use like factories or offices, or residential use like public or private housing.

### Using AlphaEarth to Classify Land-Use

As the Satellite Embedding Dataset has countless more dimensions to a certain plot of land than any singular full spectrum satellite image, it can classify land-use with an astonishing amount of context. Context, or the lack of it, is the singlemost challenging aspect of remote sensing. A pixel of forest carries data, but what the pixel was 10 years ago, or what the pixel is surrounded by would carry a lot more data. Meaningful data is hard to collect and clean, but the Satellite Embeddings Dataset has us covered in that avenue.

I used the Satellite Embedding Dataset's "Supervised Classification" method. This method involves using an image of Sentinel-2 data, visualised in false colour to exaggerate differences in vegetation, to allow us to lay pins down. These pins would create samples of pixels for the training of the classification model. A set of pins would be placed in grassland, and another in forests. Do this for water, concrete, and mangroves as well.

After training the model, it can be used to pinpoint similar water, concrete, mangrove, forest, and grassland pixels in your area of interest. And you're done! Now you have a model that can create Land-Use Classification maps for any place in the world, as long as the climate is similar to where the classification model is trained on.

## Getting Started
### Finding Training Samples
![Geometry Import menu of GEE](/images/blog/AlphaEarth/GeometryImport.png){: .limited-height}

First, we have to find some good examples of the types of land-use we wish to classify. Above is an image of my completed table of ten categories. For every category of land-use, we will create a new geometry. This can be done by clicking the "+ new layer" button under "Geometry Imports".

![Geometry Import menu (FeatureCollection)](/images/blog/AlphaEarth/GeometryImportFeatureCollection.png){: .limited-height}

Make sure to select "Feature Collection".

![Geometry Import menu (Properties)](/images/blog/AlphaEarth/GeometryImportProperties.png){: .limited-height}

And then add "landcover" as a <strong>property name</strong>, and a number from 1 to however land-use categories you have. Use "10" if you have ten categories. Also ensure you are drawing with "Point Drawing" instead of "Polygon" or "Rectangle Drawing".

![Geometry Import menu (Examples)](/images/blog/AlphaEarth/ExamplesOfCategories.png){: .limited-height}

For each category, we need to place pins on the good examples of their corresponding category. Below, there are white pins placed on areas designated as Scrubland, green pins placed on areas designated as Dipterocarp Forest, and yellow pins placed on areas designated as Urban Vegetation. These pins will be used as a sample to feed the machine learning algorithm.

We can start placing pins after the Sentinel-2 False Colour projection has been completed.

### Area of Interest
You can choose two methods to highlight your area of interest (aoi). The first method is to use a pre-existing dataset of rasters of every country's coastline and border.

{% highlight ruby %}
//Load FAO Country Borders Dataset as AREA OF INTEREST
var aoi = ee.FeatureCollection("FAO/GAUL/2015/level0").filter(ee.Filter.inList('ADM0_NAME', 
                        [
                         'Malaysia',
                         'Singapore',
                         'Brunei Darussalam'
                         ]));
{% endhighlight %}

The second method is to simply draw a rectangle with GEE's interface, as done previously in this tutorial. But this time, you can use "Polygon Drawing" or "Rectangle Drawing" instead of "Point Drawing".

{% highlight ruby %}
Map.centerObject(geometry, 4); // The smaller the number, the more zoomed out.
// Map.centerObject(aoi, 4); // The smaller the number, the more zoomed out.
{% endhighlight %}

We will now centre the aoi or geometry. Adjust the zoom scale number as needed.

### Choosing Our Time Period of Interest

Now, we have to choose our time period where we wish to draw data from to conduct our land-use classification. As the Satellite Embedding Dataset also includes a temporal scale, multiple years of data will be taken.

{% highlight ruby %}
// Pick a year for classification
var year = 2020;
var startDate = ee.Date.fromYMD(year, 1, 1);
var endDate = startDate.advance(1, 'year');
{% endhighlight %}

## Visualisation of Sentinel-2 Data
We will now look back and draw Sentinel-2 data with a set of code explained in a prior blog post. But now the visualisation will be in a False Colour palette. This means that the colour palette will no longer be RGB, but SWIR1, NIR, and Red. This would increase the amount of contrast between areas like mangrove forests, grasslands, urban areas, and terrestrial forests. 

{% highlight ruby %}
// Create a Sentinel-2 composite for the selected year
// for selecting training samples
var s2 = ee.ImageCollection('COPERNICUS/S2_SR_HARMONIZED');
var filteredS2 = s2
  .filter(ee.Filter.date(startDate, endDate))
  .filter(ee.Filter.bounds(geometry));

// Use the Cloud Score+ collection for cloud masking
var csPlus = ee.ImageCollection('GOOGLE/CLOUD_SCORE_PLUS/V1/S2_HARMONIZED');
var csPlusBands = csPlus.first().bandNames();
var filteredS2WithCs = filteredS2.linkCollection(csPlus, csPlusBands);

function maskLowQA(image) {
  var qaBand = 'cs';
  var clearThreshold = 0.6;
  var mask = image.select(qaBand).gte(clearThreshold);
  return image.updateMask(mask);
}

var filteredS2Masked = filteredS2WithCs
  .map(maskLowQA)
  .select('B.*');
{% endhighlight %}

This is the component where the False Colour bands are chosen, and the "Map.addLayer" component is the line that actually adds it to the map on the bottom of the screen.

{% highlight ruby %}
// Create a median composite of cloud-masked images
var composite = filteredS2Masked.median();
// Display the input composite
var swirVis = {min: 300, max: 4000, bands: ['B11', 'B8', 'B4']};

Map.addLayer(composite.clip(geometry), swirVis, 'S2 Composite (False Color)');
{% endhighlight %}

## Classifier Trainer
This portion will be the code that will get the machine-learning algorithm to look at your training samples, which you will add after the first run.

{% highlight ruby %}
var embeddings = ee.ImageCollection('GOOGLE/SATELLITE_EMBEDDING/V1/ANNUAL');

var embeddingsFiltered = embeddings
  .filter(ee.Filter.date(startDate, endDate))
  .filter(ee.Filter.bounds(geometry));

var embeddingsImage = embeddingsFiltered.mosaic();

// Overlay the samples on the image to get training data.
var training = embeddingsImage.sampleRegions({
  collection: gcps,
  properties: ['landcover'],
  scale: 10
});

print('Training Feature', training.first());

var classifier = ee.Classifier.smileKNN().train({
  features: training,
  classProperty: 'landcover',
  inputProperties: embeddingsImage.bandNames()
});

var classified = embeddingsImage.classify(classifier);
// Classify the embeddings mosaic
{% endhighlight %}

## Visualising the Land-Use Classification Output
Now we will visualise the output of the machine-learning algorithm. It has already looked at your training samples, and has searched for other areas to categorise into your however many categories of land cover or land-use.

However many categories of land-use do you have? That number will be the amount of colours you need. As I have 10 categories, there are 10 corresponding colours in the code sample below.

{% highlight ruby %}
// Choose a 3-color palette
// Assign a color for each class in the following order
var palette = [
  'brown', // Mangrove
  'blue', // Water
  'gray', // Concrete
  'green', // Dipterocarp Forest
  'orange', // Grassland
  'white', // Scrubland
  'purple', // Secondary Forest
  'yellow', // Urban or Managed Vegetation
  'black', // Farmland / Oil Palm Plantations
  'red']; //Paddyfields
{% endhighlight %}

Now we will add the visualisation layer to the map.
{% highlight ruby %}
Map.addLayer(
  classified.clip(geometry),
  {min: 1, max: 10, palette: palette},
  'Classified Satellite Embeddings Image');


var classifier = ee.Classifier.smileKNN().train({
  features: training,
  classProperty: 'landcover',
  inputProperties: embeddingsImage.bandNames()
});

var classified = embeddingsImage.classify(classifier);
{% endhighlight %}

## Adding the Training Samples
Now, you can add your pins to the different FeatureCollections for each of your categories of land-use. Re-run the program and watch the entire map colour up into their land-use classifications!

You can also move your aoi to another region and re-run the script. It will just classify the new aoi, but beware that it might not be factual if the training samples are from a very different climate or region!

## Exporting To Google Drive
Now check that your output makes sense. If it does, and you wish to save it, you can download it into your Google Drive if it has enough space.

{% highlight ruby %}
// Export a cloud-optimized GeoTIFF.
Export.image.toDrive({
  image: classified,
  description: 'Land-Use classification using AlphaEarth',
  crs: 'EPSG:3857',
  region: geometry,
  fileFormat: 'GeoTIFF',
  formatOptions: {
    cloudOptimized: true
  }
});
{% endhighlight %}