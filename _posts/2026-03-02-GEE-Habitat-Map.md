---
layout: post
title: Detecting Changes in Classes of Land-Use in Singapore from 2017 to 2025
categories: GIS
tags: GIS AlphaEarth Remote-Sensing
---

In this blog post, we cover some line graphs made using Google Earth Engine and the Satellite Embeddings dataset, which we have tried using in a previous blog post.

## Some Context
### Land Use Classification

I was doing some LUC for the ASEAN region, so oil palm plantations and rice paddyfields were included as two out of ten classes. But in Singapore, there are no paddyfields or plantations. So you can ignore the two classes. I will be using LUC as an abbreviation for Land-Use Class or Land-Use Classification.

### Time Period

The Satellite Embeddings dataset only starts from 2017. That means if we wanted to go back further in time, we have to revert to using conventional methods like the Normalised Difference Vegetation Index (NDVI), and using different sensors like the Landsat's Multi Spectral Scanner (MSS), and Thematic Mapper (TM).

So for this post, we will only use the Satellite Embeddings dataset, and thus data from 2017 to 2025 will be used.

## Graphs

We can now look at some LUC graphs, which is why you're here!

### Absolute Percentages

<a data-flickr-embed="true" href="https://www.flickr.com/photos/204247661@N03/55124630588/in/dateposted-public/" title="All Habitat Coverage Over Time — Singapore (2017-2025)"><img src="https://live.staticflickr.com/65535/55124630588_8990c8db01_c.jpg" width="800" height="367" alt="All Habitat Coverage Over Time — Singapore (2017-2025)"/></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>

In the above chart, you can see eight lines corresponding to eight different LUCs. Concrete and Managed Vegetation are very distinctly different habitats, but they are always changing. This is possibly due to Singapore's constant redevelopment of urban spaces. Nature playgardens, carpark-to-neighbourhood-park conversions, young streetscape trees growing larger annually, there are many reasons the concrete-to-urban-vegetation ratio keeps flipping.
But as Singapore develops its land constantly, grasslands, and secondary forests, are decreasing in area and concrete areas gain.

### Percentage Change

<a data-flickr-embed="true" href="https://www.flickr.com/photos/204247661@N03/55124832760/in/dateposted-public/" title="All Habitat Coverage — Percentage Point Change from 2017 (Singapore)"><img src="https://live.staticflickr.com/65535/55124832760_faffe22a7b_c.jpg" width="800" height="367" alt="All Habitat Coverage — Percentage Point Change from 2017 (Singapore)"/></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>

Here is a different way to visualise the change in LUC percentages. Percentage change from an origin of 0% means you can no longer compare the percentages of area between the LUCs, but now you can see the amount of increase or decrease per LUC very easily.
Here you see how Concrete and Managed Vegetation are the top two most variable LUCs, and by a massive margin as well! Interestingly, the machine learning algorithm saw an increase in Mature Forest. This might be due to changes in the Central Catchment Nature Reserve (CCNR)'s conditions, thus creating an illusion of more Mature Forests, or that the CCNR is simply succeeding and increasing in maturity.