---
title: "QGIS: Batch Processing"
date: 2024-07-30T08:06:25+06:00
description: An overview on QGIS batch processing capabilities
menu:
  sidebar:
    name: Batch Processing
    identifier: batch_processing
    parent: qgis
    weight: 1
---

## Project Overview
In this tutorial, we'll explore the capabilities of batch processing in QGIS. Batch processing is incredibly helpful for repetitive tasks that can otherwise consume a lot of time and effort. By automating these tasks, you can focus on more important aspects of your project, increasing both efficiency and productivity.

Let me set up an example where we'll fully leverage the power of batch processing in QGIS.

## Case Example
We've got a set of 10 MODIS land surface temperature (LST) images for an area of interest. These images are in Kelvin degrees, but before we can perform our analysis in Celsius, we need to first scale the temperature values by a factor of 0.01. After scaling, we'll convert the temperatures from Kelvin to Celsius by subtracting 273.

If we were to take the regular approach, we would have to open the Raster Calculator in QGIS and perform these operations 12 times, once for each image. However, with QGIS's batch processing tool, we can automate these tasks and handle all 12 images efficiently in one go.

First we'll load the images into QGIS as shown in the picture below:

{{< vs 1 >}}

{{<img src="images/_01.png" align="center">}}

{{< vs 1 >}}

To make things simpler, I'll rename layers and number them (according to the date):

{{< vs 1 >}}

{{<img src="images/_02.png" align="center">}}

{{< vs 1 >}}

To perform the scaling and conversion, we will use the Raster Calculator. Note that the Raster Calculator in the traditional menu doesn’t support batch processing, so we need to access it through the Processing Toolbox as shown below:

{{< vs 1 >}}

{{<img src="images/_03.png" align="center">}}

{{< vs 1 >}}

A window will open up and we will first click on the option *'Run as Batch Process'*. Afterwards, we’ll see the following menu:

{{< vs 1 >}}

{{<img src="images/_04.png" align="center">}}

{{< vs 1 >}}

On this menu, we’ll need to fill out a table with the arguments for the process we are applying to each of our layers. We'll describe the necessary arguments for each of these columns:

- **Reference layer:** let’s start by filling up the *'Reference layer'* column. If we click on the *'Select from open layers'* option, we’ll be able to choose from a list of all the available layers. As a result, the table will be automatically populated with these layers.

- **Expression:** Since we have numbered the layers, we will define the expression for the first layer, then use the fill down option for the other layers. Afterwards, manually edit each expression to match the corresponding layer name (this is straightforward as we have numbered the layers).

- **Cell size:** By consulting layers metadata, we see that the spatial resolution is of 925 meters. We’ll use this value and populate each row by using the fill down option again.

- **Output extent:** For the first layer we'll click on the three dots and select the *'Calculate from layer option'*. Remaining layers we'll be populated with the same value by using the fill down option.

- **Output CRS:** we'll choose the same CRS of the original layers and project and once again fill down to every other row.

- **Output:** Finally, it only remains to define the output for each layer. Click on the three dots for the first layer, set up a prefix (such as an underscore), and then click on the *'Fill with numbers'* option.

As a result of all the steps we've described above, the completed table should look like the picture shown below:

{{< vs 1 >}}

{{<img src="images/_05.png" align="center">}}

{{< vs 1 >}}

Dont forget to check the *'Load layers on completion'* option to visualize your result upon completion!
We are now set up to execute the batch process. Click on Run, and if everything works correctly, no warnings should appear in the console. As a result, new layers should appear in the Layers panel, containing the converted temperatures from Kelvin to Celsius.

## Conclusion
Batch processing might seem daunting at first, but once you get the hang of it, it can be incredibly useful. Understanding the logic behind setting it up will make you more effective and help reduce repetitive and exhausting tasks. I hope this tutorial has been helpful. Stay tuned for more tutorials!