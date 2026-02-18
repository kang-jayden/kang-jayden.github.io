---
layout: post
title: Utilising Google's AlphaEarth in Classification of Land Use
categores: GIS
tags: GIS AlphaEarth
---

## Some Context
### What is AlphaEarth?

Google Earth Engine and Google DeepMind came together and created a new dataset known as the Satellite Embedding Dataset. There are 64 embeddings created by running every single LandSat and Sentinel-2 image through a machine learning algorithm, which then tagged every pixel to 64 dimensions or properties. Examples of these dimensions could be temporal, temperature, and reflectance of the staple few electromagnetic bands.

### What is Land-Use Classification?

Land-Use Classification is the category of activity occuring on a certain land parcel. It could be slated for educational use, commercial use like factories or offices, or residential use like public or private housing.

## Using AlphaEarth to Classify Land-Use

As the Satellite Embedding Dataset has countless more dimensions to a certain plot of land than any singular full spectrum satellite image, it can classify land-use with an astonishing amount of context. Context, or the lack of it, is the singlemost challenging aspect of remote sensing. A pixel of forest carries data, but what the pixel was 10 years ago, or what the pixel is surrounded by would carry a lot more data. Meaningful data is hard to collect and clean, but the Satellite Embeddings Dataset has us covered in that avenue.


I used the Satellite Embedding Dataset's "Supervised Classification" method. This method involves using an image of Sentinel-2 data, visualised in false colour to exaggerate differences in vegetation, to allow us to lay pins down. These pins would create samples of pixels for the training of the classification model. A set of pins would be placed in grassland, and another in forests. Do this for water, concrete, and mangroves as well.


After training the model, it can be used to pinpoint similar water, concrete, mangrove, forest, and grassland pixels in your area of interest. And you're done! Now you have a model that can create Land-Use Classification maps for any place in the world, as long as the climate is similar to where the classification model is trained on.