---
layout: post
title: Mapping Out Dipterocarp Forests with Remote Sensing
---
# Cloud Masking of Sentinel-2 Data
First, we need clean data to work with. If we were using only the Normalised Difference Vegetation Index (NDVI), then using Copernicus' Sentinel-2 Quarterly Global Mosaics would be of no issue. But, as we are also creating a Normalised Difference Moisture Index (NDMI), which requires Band 11 (SWIR2), we can't use the Quarterly Global Mosaics.


This means that we need to create our own cloudless images to work with.

# Creation of NDVI
Creation of the NDVI is very straightforward.

# Creation of NDMI
In the raster calculator, we can now look into creating the NDMI with our cloudless mosaic.

# Merging NDVI with NDMI
Now we will head to the raster calculator