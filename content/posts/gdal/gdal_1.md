---
title: "Calculate slopes from a DEM using GDAL and Python"
date: 2024-09-27T00:00:00+00:00
description: Calculate slopes from a DEM using GDAL and Python
menu:
  sidebar:
    name: Slopes with GDAL
    identifier: gdal_slopes
    parent: gdal
    weight: 1
---

## Project Overview

In this post, we'll explore the capabilities of GDAL tools alongside Python to automate the process and calculate the mean slope of multiple basins from a Digital Elevation Model (DEM).

To begin with, we'll need to set up an Anaconda environment with GDAL installed in it. I strongly recommend following the steps detailed in this [tutorial.](https://courses.spatialthoughts.com/gdal-tools.html#setting-up-the-environment)

The project is reproducible and available on my (Github repository)[https://github.com/jm-marcenaro/Personal-blog-posts], feel free to visit!

## Hands on the code
Once we've got our environment set up, lets move forward with the code. We'll start by importing the necessary libraries and defining the path to de DEM and Shapefile containing the basins from where we want to extract the mean slope.

{{< vs 1 >}}
```python
# Libraries
import os
import subprocess
import re

# Path to DEM file
DEM = r"DEMs/dem_01.tif"

# Path to shapefile
SHP = r"SHPs/basins_01.shp"
```
{{< vs 1 >}}

Next up, we'll define a list with the basins name or ID. This information can be exctracted from the dbf file.

{{< vs 1 >}}
```python
# List of basins ID
Bs = [
"B-01", "B-02", "B-03", "B-04", "B-05", "B-06",
"B-07","B-08", "B-09", "B-10", "B-11", "B-12",
"B-13", "B-14", "B-15", "B-16", "B-17", "B-18", "B-19"
]
```
{{< vs 1 >}}

We are now ready to create our workflow, which will consist of the following steps:

1. **Clip the DEM:** For each basin polygon, we'll clip the DEM and save it as a `.tif` file.
2. **Compute Slope:** Using the clipped DEM, we'll calculate the slope and write the output to a new `.tif` file.
3. **Calculate Statistics:** From each slope file, we will compute statistical values and save them to a `.txt` file.
4. **Extract Mean Slope:** Using Python regular expressions, we'll read each of these `.txt` files and extract the mean slope value.
5. **Store and Summarize:** We'll store the mean slope value of each basin in a dictionary. Once the loop is completed, a final `.txt` file will summarize the mean slope for all basins.

The code will look like this:

{{< vs 1 >}}
```python
# Empty dictionary to store outputs
DICT_SLs = {}

# Iterate over each basin
for B in Bs:

    print(f"{B}:")
    
    print("- Clipping DEM.")
            
    # Path to output file
    OUT_1 = os.path.join('Output', f'DEM_{B}.tif')
    
    # Generate the command to clip
    CMD_2 = f"{CMD_1} && gdalwarp -overwrite -of GTiff -cutline {SHP} -cwhere \"ID_1='{B}'\" -dstnodata -9999 -crop_to_cutline {DEM} {OUT_1}"
    
    # Execute the command
    subprocess.run(CMD_2, stdout=subprocess.DEVNULL)

    # Calculate slopes from clipped DEM

    print(f"- Calculating slope.")
    
    # Path to output file
    OUT_2 = os.path.join('Output', f'SL_{B}.tif')

    # Generate the command to calculate slope
    CMD_3 = f"{CMD_1} && gdaldem slope {OUT_1} {OUT_2} -of GTiff -b 1 -s 1.0 -p"

    # Execute the command
    subprocess.run(CMD_3, stdout=subprocess.DEVNULL)

    # Read slope file and compute statistics over it.
    print(f"- Analysing slope file.")
    
    # Path to output file
    OUT_3 = os.path.join('Output', f'SL_{B}.txt')
    
    # Generate the command to calculate statistics
    CMD_4 = f"{CMD_1} && gdalinfo -stats {OUT_2} > {OUT_3}"
    
    # Execute the command
    subprocess.run(CMD_4, stdout=subprocess.DEVNULL)

    # Open txt file and extract mean slope
    with open(OUT_3, 'r') as file:
        
        TXT = file.read()

    # Regular expression to find the value of STATISTICS_MEAN
    SL_1 = re.findall(r"STATISTICS_MEAN=([\d.]+)", TXT)

    SL_2 = float(SL_1[0])
    
    print(f"- Mean slope: {SL_2:.2f} %\n")

    # Store value in dictionary
    DICT_SLs[f"{B}"] = SL_2

# Write dicitionary to a txt file

with open(os.path.join("Output", "SLs.txt"), 'w') as F:
    
    F.write(str(DICT_SLs))
    print("Created SLs.txt containing the mean slope of each basin.")
```
{{< vs 1 >}}

Let's get into more detail with some aspects of the code:
- The variable `CMD_2` constructs the GDAL command to clip the DEM to each polygon from the shapefile. In each iteration of the `for` loop it is filtering by the `ID_1` field from the shapefile.
- The variable `CMD_3` constructs the GDAL command to calculate the slope from each clipped DEM. Options `-b 1 -s 1.0 -p` are declaring that we'll use the first band from the `tif` file, that the aspect ratio is equal to 1 and that the output should be expressed as percentage rather than degrees.

## Conclusion

In this post, we explored how to calculate the mean slope of multiple basins from a DEM using GDAL commands and Python to automate the process.

The possibilities for geospatial analysis are vast, and mastering these tools can significantly enhance your data analysis capabilities. In my case, I've utilized this workflow to facilitate the process of calculating the concentration time of multiple basins in my hydrological analysis.

Feel free to adapt this code for your own projects. Thank you for reading, and I hope you found this tutorial helpful!