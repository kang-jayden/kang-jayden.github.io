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

![Geometry Import menu of GEE](/images/blog/AlphaEarth/GeometryImport.png){: .limited-height}

First, we have to find some good examples of the types of land-use we wish to classify. Above is an image of my completed table of ten categories. For every category of land-use, we will create a new geometry. This can be done by clicking the "+ new layer" button under "Geometry Imports".

![Geometry Import menu of GEE](/images/blog/AlphaEarth/GeometryImportFeatureCollection.png){: .limited-height}

Make sure to select "Feature Collection".

![Geometry Import menu of GEE](/images/blog/AlphaEarth/GeometryImportProperties.png){: .limited-height}

And then add "landcover" as a <strong>property name</strong>, and a number from 1 to however land-use categories you have. Use "10" if you have ten categories.

![Geometry Import menu of GEE](/images/blog/AlphaEarth/ExamplesOfCategories.png){: .limited-height}

For each category, we need to place pins on the good examples of their corresponding category. Below, there are white pins placed on areas designated as Scrubland, green pins placed on areas designated as Dipterocarp Forest, and yellow pins placed on areas designated as Urban Vegetation. These pins will be used as a sample to feed the machine learning algorithm.
