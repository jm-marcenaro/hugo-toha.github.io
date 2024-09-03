---
title: "GEE Python API: NDVI, precipitation and real evapotranspiration"
date: 2024-09-02T08:06:25+06:00
description: An overview on NDVI, precipitation and real evapotranspiration using GEE Python API.
menu:
  sidebar:
    name: NDVI, Ppt and ETr
    identifier: ndvi-ppt-etr-1
    parent: gee
    weight: 3
---

### Project Overview
In this post, we'll explore the correlation between multiple environmental data variables using the Google Earth Engine (GEE) Python API. Specifically, we'll analyze yearly aggregated Normalized Difference Vegetation Index (NDVI), precipitation, and real evapotranspiration (ETr) over a region of interest spanning approximately 5,000 km² and over a five-year period (2019-2023).

- NDVI data will be extracted from the Sentinel-2 satellites.
- Precipitation data will be sourced from the CHIRPS dataset.
- Real evapotranspiration (ETr) data will be obtained from the MODIS satellite.

To make this project reproducible, you can access all the code from my [GitHub repository](https://github.com/jm-marcenaro/hugo-posts). Feel free to check it out, try the code yourself, and leave comments or suggestions.

### Analysis
To begin with, we'll import the necessary libraries, initialize earth engine and define our region of interest by reading a shapefile and extracting its bounding box with Geopandas. The region corresponds to a department within Buenos Aires province, Argentina.


{{< vs 1 >}}
```python
# Libraries
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
from matplotlib.colors import ListedColormap
import matplotlib.colors as colors
import geopandas as gpd
from datetime import datetime, timedelta
import ee
import urllib.request
import rasterio
from rasterio.mask import mask
import xarray as xr

# Initialize EE
ee.Initialize()

# Read shapefile with geopandas
GDF_01 = gpd.read_file(r"GIS/ROI.shp")

# For now, we'll work with ROI´s bounding box
BB_01 = GDF_01.total_bounds

# Define coordinates
x_min, y_min, x_max, y_max = BB_01

# Create ee.Geometry
ROI = ee.Geometry.Rectangle([x_min, y_min, x_max, y_max])
```
{{< vs 1 >}}

Next, we'll retrieve NDVI data using Sentinel-2’s near-infrared (Band 8) and red (Band 4) bands. We’ll apply a threshold to exclude pixels with more than 25% cloud coverage. The data will cover a 5-year period, from 2019 to 2023.

```python
# Define years of interest
Ys = [2019, 2020, 2021, 2022, 2023]

# Define a dictionary of image collections
DICT_Cs_01 = {}

for Y in Ys:

    DICT_Cs_01[f"{Y}"] = ee.ImageCollection('COPERNICUS/S2_SR_HARMONIZED')\
                .filterBounds(ROI)\
                .filterDate(f"{Y}-01-01", f"{Y+1}-01-01")\
                .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 25))
```
{{< vs 1 >}}

NDVI is not directly available as a band; instead, we need to calculate it by taking the normalized difference between the bands mentioned above. We’ll define a function to compute NDVI and then map it over the image collections we've defined.

{{< vs 1 >}}
```python
# Calculate NDVI on every image

# Create a function to calculate NDVI
def s2_ndvi(image):
    
    ndvi = image.normalizedDifference(['B8', 'B4']).rename('NDVI')
    
    return image.addBands(ndvi)

# Map the function over the ICs
for k, C in DICT_Cs_01.items():
    
    DICT_Cs_01[f"{k}"] = C.map(s2_ndvi)
```
{{< vs 1 >}}

Since we are aggregating the data on a yearly basis, we will use the median NDVI value for each pixel. The median is preferred over the mean because it is less sensitive to outliers and provides a more accurate measure of central tendency. To facilitate this, we’ll define a dictionary of images that computes the median NDVI value for each pixel for each year in the analysis.

{{< vs 1 >}}
```python
# For each year create an image that takes the median NDVI

# Dicitonary of images
DICT_Is_01 = {}

for k, C in DICT_Cs_01.items():
    
    DICT_Is_01[f"{k}"] = C.select('NDVI').median()
```
{{< vs 1 >}}

Now we'll download the images locally so that we can read them afterwards with the library Rasterio. This library will enable us to compute statistics over the images and the ability to create figures out of them. Note that we used a 500-meter scale, as the original Sentinel-2 spatial resolution of 10 meters would have required significantly more memory.

{{< vs 1 >}}

```python
# Download images locally

for k, I in DICT_Is_01.items():

  FN = f"S2-MED-NDVI-{k}.tif"

  # Define the export parameters.
  url = I.getDownloadURL({
  "scale" : 500,
  "crs" : "EPSG:4326",
  "region" : ROI,
  "filePerBand": False,
  "format" : "GEO_TIFF"
  })

  # Start downloading
  urllib.request.urlretrieve(url, fr"Output/{FN}")
  print(f"{FN} downloaded!")
```
{{< vs 1 >}}

We will now read the images using Rasterio, define a custom NDVI color palette, and set the parameters for our figures.

```python
# Read images with rasterio
DICT_Rs_01 = {}

for Y in Ys:

    FN = f"S2-MED-NDVI-{Y}.tif"

    DICT_Rs_01[f"{Y}"] = rasterio.open(fr"Output/{FN}")

# Create NDVI palette

ndvi_palette_hex = ['FFFFFF', 'CE7E45', 'DF923D', 'F1B555', 'FCD163', '99B718', '74A901', '66A000',
               '529400', '3E8601', '207401', '056201', '004C00', '023B01', '012E01', '011D01', '011301'] 

ndvi_palette_rgba  = [colors.hex2color('#' + hex_color) for hex_color in ndvi_palette_hex]

ndvi_palette = ListedColormap(ndvi_palette_rgba)

# Figure parameters

# Width, height
W, H = 4, 2.5

# Aspect ratio
AR = W/H

# Set figure bounds
Bs = DICT_Rs_01["2019"].bounds

# Set fontsizes
T_FS = 14
AX_FS = 12
```
{{< vs 1 >}}

We are now ready to use Matplotlib and create our first figure:

{{< vs 1 >}}

```python
fig, axs = plt.subplots(1, 5, figsize=(5*W, H), constrained_layout=True)

for (k, R), ax in zip(DICT_Rs_01.items(), axs.ravel()):

    _ = ax.imshow(R.read(1), vmin=0.25, vmax=0.75, cmap=ndvi_palette, aspect=1/AR, extent=(Bs.left, Bs.right, Bs.bottom, Bs.top))

    ax.set_title(f"Median NDVI {k}", fontweight="bold", fontsize=T_FS)

    GDF_01.plot(ax=ax, facecolor="none", edgecolor="black", zorder=2, linewidth=2, alpha=1)
    
    ax.axis("off")

CB = plt.colorbar(_, extend="both", ticks=np.arange(0.25, .80, .05), shrink=.75)

CB.ax.tick_params(labelsize=AX_FS)

CB.set_label("[%]", fontsize=AX_FS)

plt.savefig(r"Output/_01.png")
plt.show();
```
{{< vs 1 >}}

{{<img src="images/_01.png" align="center">}}

{{< vs 1 >}}

We are now ready to move on to our next variable: precipitation. For this, we will use the CHIRPS product, which provides daily precipitation values with a spatial resolution of approximately 5000 meters.

Similarly to what we did before, we'll define a dictionary of image collections, one for each year.

{{< vs 1 >}}

```python
# Define a dictionary of image collections
DICT_Cs_02 = {}

for Y in Ys:

    DICT_Cs_02[f"{Y}"] = ee.ImageCollection("UCSB-CHG/CHIRPS/DAILY")\
                .filterBounds(ROI)\
                .filterDate(f"{Y}-01-01", f"{Y+1}-01-01")
```
{{< vs 1 >}}

Since we are aggregating data yearly, we will compute the cumulative sum of precipitation at pixel level. To accomplish this, we'll define a new dictionary of images that calculates the annual sum for each pixel and year.

{{< vs 1 >}}
```python
# Define new dictionary of images with the cumulative sum for each year
DICT_Is_02 = {}

for k, C in DICT_Cs_02.items():

    DICT_Is_02[k] = C.sum()
```
{{< vs 1 >}}

As we did before, we'll download images locally. This time we'll use the original spatial resolution.

{{< vs 1 >}}
```python
# Get spatial resolution
OS_02 = DICT_Cs_02["2019"].select("precipitation").first().projection().nominalScale().getInfo()

# Download images locally

for k, I in DICT_Is_02.items():

  FN = f"CHIRPS-AC-PPT-{k}.tif"

  # Define the export parameters.
  url = I.getDownloadURL({
  "scale" : OS_02,
  "crs" : "EPSG:4326",
  "region" : ROI,
  "filePerBand": False,
  "format" : "GEO_TIFF"
  })

  # Start downloading
  urllib.request.urlretrieve(url, fr"Output/{FN}")
  print(f"{FN} downloaded!")
```
{{< vs 1 >}}

Once we've written the images to disk we'll read them with Rasterio as we previously did.

{{< vs 1 >}}

```python
# Read images with rasterio
DICT_Rs_02 = {}

for Y in Ys:

    FN = f"CHIRPS-AC-PPT-{Y}.tif" 

    DICT_Rs_02[f"{Y}"] = rasterio.open(fr"Output/{FN}")
```

{{< vs 1 >}}

And we are now ready to use Matplotlib and create our second figure:

{{< vs 1 >}}

```python
fig, axs = plt.subplots(1, 5, figsize=(5*W, H), constrained_layout=True)

for (k, R), ax in zip(DICT_Rs_02.items(), axs.ravel()):

    _ = ax.imshow(R.read(1), vmin=500, vmax=1000, cmap="Blues", aspect=1/AR, extent=(Bs.left, Bs.right, Bs.bottom, Bs.top))

    ax.set_title(f"Cumulated PPT {k}", fontweight="bold", fontsize=T_FS)

    GDF_01.plot(ax=ax, facecolor="none", edgecolor="black", zorder=2, linewidth=2, alpha=1)
    
    ax.axis("off")

CB = plt.colorbar(_, extend="both", ticks=np.arange(500, 1050, 50), shrink=.75)

CB.ax.tick_params(labelsize=AX_FS)

CB.set_label("[mm]", fontsize=AX_FS)

plt.savefig(r"Output/_02.png")
plt.show();
```
{{< vs 1 >}}

{{<img src="images/_02.png" align="center">}}

{{< vs 1 >}}

Finally, we'll get real evapotranspiration values from MODIS. See more on this product [here](https://developers.google.com/earth-engine/datasets/catalog/MODIS_061_MOD16A2GF).

We'll start by defining a dictionary of image collections. Note we've scaled data by 0.1 as indicated in the documentation.

{{< vs 1 >}}
```python
# Define a dictionary of image collections
DICT_Cs_03 = {}

for Y in Ys:

    DICT_Cs_03[f"{Y}"] = ee.ImageCollection("MODIS/061/MOD16A2GF")\
                .filterBounds(ROI)\
                .filterDate(f"{Y}-01-01", f"{Y+1}-01-01")\
                .map(lambda i: i.select('ET').multiply(0.1))
```
{{< vs 1 >}}

Evapotranspiration values are provided as cumulative sums over 8-day periods. Since we are aggregating the data on a yearly basis, we will calculate the cumulative sum of evapotranspiration for each year. To achieve this, we'll define another dictionary of images, just as we did for the other variables.

{{< vs 1 >}}
```python
# For each year create an image that takes the sum of ET

# Dictionary of images
DICT_Is_03 = {}

for k, C in DICT_Cs_03.items():
    
    DICT_Is_03[f"{k}"] = C.select('ET').sum()
```
{{< vs 1 >}}

As we did before, we'll use the product's spatial resolution, which is approximately 500 meters, and download the images locally.

{{< vs 1 >}}
```python
# Get spatial resolution
OS_03 = DICT_Cs_03["2019"].select("ET").first().projection().nominalScale().getInfo()

# Download images locally

for k, I in DICT_Is_03.items():

  FN = f"MODIS-ETR-{k}.tif"

  # Define the export parameters.
  url = I.getDownloadURL({
  "scale" : OS_03,
  "crs" : "EPSG:4326",
  "region" : ROI,
  "filePerBand": False,
  "format" : "GEO_TIFF"
  })

  # Start downloading
  urllib.request.urlretrieve(url, fr"Output/{FN}")
  print(f"{FN} downloaded!")
```
{{< vs 1 >}}

We read images with Rasterio:

{{< vs 1 >}}
```python
# Read images with rasterio
DICT_Rs_03 = {}

for Y in Ys:

    FN = f"MODIS-ETR-{Y}.tif"

    DICT_Rs_03[f"{Y}"] = rasterio.open(fr"Output/{FN}")
```

{{< vs 1 >}}

And we plot them using Matplotlib:

{{< vs 1 >}}

```python
fig, axs = plt.subplots(1, 5, figsize=(5*W, H), constrained_layout=True)

for (k, R), ax in zip(DICT_Rs_03.items(), axs.ravel()):

    _ = ax.imshow(R.read(1), vmin=250, vmax=1000, cmap="Greens", aspect=1/AR, extent=(Bs.left, Bs.right, Bs.bottom, Bs.top))

    ax.set_title(f"Cumulated ETR {k}", fontweight="bold", fontsize=T_FS)

    GDF_01.plot(ax=ax, facecolor="none", edgecolor="black", zorder=2, linewidth=2, alpha=1)
    
    ax.axis("off")

CB = plt.colorbar(_, extend="both", ticks=np.arange(200, 1100, 100), shrink=.75)

CB.ax.tick_params(labelsize=AX_FS)

CB.set_label("[mm]", fontsize=AX_FS)

plt.savefig(r"Output/_03.png")
plt.show();
```
{{< vs 1 >}}

{{<img src="images/_03.png" align="center">}}

{{< vs 1 >}}

We are now ready to summarize all our data and visualize it together to examine how these variables correlate and the impact they had each year.

```python
fig, axs = plt.subplots(3, 5, figsize=(5*W, 3*H), constrained_layout=True)

for (k, R), ax in zip(DICT_Rs_01.items(), axs.ravel()[:5]):

    _ = ax.imshow(R.read(1), vmin=0.25, vmax=0.75, cmap=ndvi_palette, aspect=1/AR, extent=(Bs.left, Bs.right, Bs.bottom, Bs.top))

    ax.set_title(f"Median NDVI {k}", fontweight="bold", fontsize=T_FS)

    GDF_01.plot(ax=ax, facecolor="none", edgecolor="black", zorder=2, linewidth=2, alpha=1)

    ax.axis("off")

    # Annotate with mean value
    ax.text(0.1, 0.1, f"{R.read(1).mean():.2f} %", transform=ax.transAxes, ha="center", va="center", color="black", fontsize=10, bbox=dict(facecolor='pink', alpha=0.75))


CB = plt.colorbar(_, extend="both", ticks=np.arange(0.25, .80, .05), shrink=.75)

CB.ax.tick_params(labelsize=AX_FS)

CB.set_label("[%]", fontsize=AX_FS)

for (k, R), ax in zip(DICT_Rs_02.items(), axs.ravel()[5:10]):

    _ = ax.imshow(R.read(1), vmin=500, vmax=1000, cmap="Blues", aspect=1/AR, extent=(Bs.left, Bs.right, Bs.bottom, Bs.top))

    ax.set_title(f"Cumulated PPT {k}", fontweight="bold", fontsize=T_FS)

    GDF_01.plot(ax=ax, facecolor="none", edgecolor="black", zorder=2, linewidth=2, alpha=1)
    
    ax.axis("off")

    # Annotate with mean value
    ax.text(0.1, 0.1, f"{R.read(1).mean():.0f} mm", transform=ax.transAxes, ha="center", va="center", color="black", fontsize=10, bbox=dict(facecolor='pink', alpha=0.75))


CB = plt.colorbar(_, extend="both", ticks=np.arange(500, 1050, 50), shrink=.75)

CB.ax.tick_params(labelsize=AX_FS)

CB.set_label("[mm]", fontsize=AX_FS)

for (k, R), ax in zip(DICT_Rs_03.items(), axs.ravel()[10:15]):

    _ = ax.imshow(R.read(1), vmin=250, vmax=1000, cmap="Greens", aspect=1/AR, extent=(Bs.left, Bs.right, Bs.bottom, Bs.top))

    ax.set_title(f"Cumulated ETR {k}", fontweight="bold", fontsize=T_FS)

    GDF_01.plot(ax=ax, facecolor="none", edgecolor="black", zorder=2, linewidth=2, alpha=1)
    
    ax.axis("off")
    
    # Annotate with mean value
    ax.text(0.1, 0.1, f"{R.read(1).mean():.0f} mm", transform=ax.transAxes, ha="center", va="center", color="black", fontsize=10, bbox=dict(facecolor='pink', alpha=0.75))


CB = plt.colorbar(_, extend="both", ticks=np.arange(200, 1100, 100), shrink=.75)

CB.ax.tick_params(labelsize=AX_FS)

CB.set_label("[mm]", fontsize=AX_FS)

plt.savefig(r"Output/_04.png")
plt.show();
```
{{< vs 1 >}}

{{<img src="images/_04.png" align="center">}}

{{< vs 1 >}}

Note we've added the main value over the region as text in the lower left corner. Anyway it would be more informative if we see can see it this way:

{{< vs 1 >}}

{{<img src="images/_05.png" align="center">}}

{{< vs 1 >}}

## Conclusion
As observed, variations in precipitation, real evapotranspiration, and NDVI tend to correlate and move together as expected. This approach not only facilitates year-over-year comparisons but also helps detect spatial trends and identify areas with varying levels of severity.

In summary, we successfully processed satellite data for NDVI, precipitation, and real evapotranspiration using various products and techniques. This analysis provided valuable insights into the region and period of interest.