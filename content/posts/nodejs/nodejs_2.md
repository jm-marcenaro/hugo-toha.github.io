---
title: "GOES Visualizer"
date: 2024-09-27T00:00:00+00:00
description: GOES application served through Node.js
menu:
  sidebar:
    name: GOES Visualizer
    identifier: goes_visualizer
    parent: nodejs
    weight: 1
---

## Project Overview
In this opportunity we'll go over a project developed through the combination of Google Earth Engine and Node.js once again. Data we'll be retrieved  and processed through GEE Python API and afterwards served to a local website interface using Node.js

We’ll walk through:
- Retrieving and processing data from Google Earth Engine using Python,
- serving this data through a local Node.js server and,
- building a responsive dashboard for data visualization.

By the end of this project, you’ll learn how to integrate these powerful tools to visualize complex geospatial datasets in real time. The project is reproducible and available on my [Github repository](https://github.com/jm-marcenaro/Visualizador-GOES) so you can follow along.

Here’s a preview of the final website interface. It features a true-color GOES animation over Argentina for a date of interest.

{{< vs 1 >}}

{{<img src="images/_01.png" align="center">}}

{{< vs 1 >}}

## Project structure

The project structure is shown below. The main directories are:
- **GEE**: Contains the Python script that retrieves and stores data from GEE.
- **public**: Contains the HTML structure, JavaScript for interactivity, and CSS for styling.

```bash
/.
├── GEE
│   ├── GIS
│   │   ├── ROI.shp       # Shapefile representing the ROI
│   │   └── Grid.shp      # Shapefile representing the grid
│   ├── GOES.py           # Python script that downloads images from GEE
│   └── Output
│       ├── PNGs          # Folder containing the downloaded and processed images
│       └── TSs.json      # File with the timestamps of the images
├── public
│   ├── index.html        # HTML structure
│   ├── ui.js             # User interface in JavaScript
│   ├── styles.css        # Stylesheet
├── images
│   ├── readme_01.png     # Images for the README
├── server.js             # Node.js server with Express.js
├── package.json          # Node.js dependencies
└── README.md             # README file
```         

## Retrieving data from GEE Python API

The Python script GOES.py downloads true-color GOES-16 satellite images over Argentina for a specific date and time of interest, plus the 6 hours preceding it and sampled every 30 minutes. It processes these images by scaling the red, blue, and near-infrared bands, creating a synthetic green band, and saves them as PNGs. Additionally, it generates a timestamp file for tracking when each image was captured. For more details on processing true-color GOES-16 images, you can refer to this [article](https://jstnbraaten.medium.com/goes-in-earth-engine-53fbc8783c16).


## Serving through a local Node.js server

The `server.js` script sets up a Node.js server using Express.js to serve the web interface. It hosts the frontend from the public directory and serves processed images from the GEE/Output/PNGs folder as well. 

It provides endpoints to list available images `/images` and timestamps `/timestamps`. Additionally, it allows end users to trigger the Python script `/run-script` that downloads and processes GOES-16 images based on a date of interest as explained before.

## Building a responsive dashboard for data visualization
The website is built using a combination of HTML, JavaScript, and CSS. Let’s break it up and explore what each component does:

#### HTML

The `index.html` file creates the structure of the web interface. It allows users to input a date and fetch GOES-16 satellite images by triggering the `GOES.py` script. It includes a text input for the date, buttons to retrieve images, and navigation controls to browse through the downloaded images.

#### Javascript
   
The `ui.js` script controls the front-end behavior. It fetches satellite images and timestamps from the server, displays them, and allows navigation through the images using "Previous" and "Next" buttons. Users can input a specific date or select the current time by clicking on "Now!". The Python script in triggered once the "Get images" button is clicked.

Besides, it disables navigation during the `GOES.py` script execution and updates the displayed image, timestamp, and image counter accordingly.

#### CSS

The CSS styles the layout centering and formatting the image display, input, and buttons. It provides responsive designs for the image container, timestamp display, and navigation buttons. It attempt to ensure an organized, visually clean interface with hover effects and a loading overlay during script execution to enhence the user experience.

## Conclusion

The project integrates a Python backend and a user-friendly frontend to display GOES-16 satellite images over Argentina allowing users to track weather and environmental changes.  Users can select specific dates, fetch images, and navigate through them with ease, offering a smooth experience.