---
title: "GFS Dashboard"
date: 2024-09-26T00:00:00+00:00
description: GFS Dashboard served through Node.js
menu:
  sidebar:
    name: GFS Dashboard
    identifier: gfs_dashboard
    parent: nodejs
    weight: 1
---

## Project Overview
In this opportunity we'll go over a project developed through the combination of Google Earth Engine and Node.js. Data we'll be retrieved  and processed through GEE Python API and afterwards served to a local dashboard using Node.js

We’ll walk through:
- Retrieving and processing data from Google Earth Engine using Python,
- serving this data through a local Node.js server and,
- building a responsive dashboard for data visualization.

By the end of this project, you’ll learn how to integrate these powerful tools to visualize complex geospatial datasets in real time. The project is reproducible and available on my [Github repository](https://github.com/jm-marcenaro/GFS-Dashboard) so you can follow along.

Here’s a preview of the final dashboard. It features a table and several time series plots that display forecast data for multiple climatic variables in the city of Buenos Aires.

{{< vs 1 >}}

{{<img src="images/_01.png" align="center">}}

{{< vs 1 >}}

## Project structure

The project structure is shown below. The main directories are:
- **DB**: Contains the Python script that retrieves and stores data from GEE, along with the database file itself.
- **public**: Contains the HTML structure, JavaScript for interactivity, and CSS for styling.

```bash
/.
│
├── DB/                     
│   ├── DBS-BD-LOG.txt       # Log file
│   ├── GFS-DB.bat           # Batch script for DB
│   ├── GFS-DB.db            # Database file
│   ├── GFS-DB.py            # Python script for DB
│   ├── GFS-DB.vbs           # VBS script for DB
│
├── images/                 
├── node_modules/           
├── public/                 
│   ├── style/               # CSS styles
│   ├── index.html           # Main HTML file
│   ├── plot.js              # JavaScript for plotting data
│   ├── table.js             # JavaScript for table functionality
│
├── package.json            
├── package-lock.json       
├── README.md               
├── server.bat              
├── server.js               # Node.js server               
├── server.vbs              # Triggers the server through the task scheduler
```         

## Retrieving data from GEE Python API

Shortly, the Python script *GFS-DB.py* retrieves the latest available weather forecast data for Buenos Aires from Google Earth Engine, processes it, and stores the results in a SQLite database. I’ve previously written more about this dataset, which you can check out [here.](https://jm-marcenaro.github.io/hugo-toha.github.io/posts/gee/gfs/)

The script starts by defining the region of interest and collects weather data (temperature, humidity, wind speed, direction, cloud cover, and precipitation) for the next 120 hours.

Using the XEE library (learn more [here](https://github.com/google/Xee)), the weather data is processed into a more usable format and organized into a pandas DataFrame. The final output is stored in a SQLite database, making it easily accessible for future use by the server.

Additionally, logging is included to track the process, and the Python script is automatically triggered every 3 hours via Windows Task Scheduler (or a cron job in Linux).

## Serving through a local Node.js server

The *server.js* script initializes an Express application to serve static files and interact with the SQLite database *GFS-DB.db* we've previously created. It then sets up a GET route at */data* to retrieve all records from the database table. Upon success, it responds with a JSON object containing the forecast data. The server listens on a specified port, providing a straightforward API for accessing this data, which can be integrated with the JavaScript code that we'll explore next.

## Building a responsive dashboard for data visualization
The dashboard is built using a combination of HTML, JavaScript, and CSS. Let’s break it up and explore what each component does:

#### HTML

The HTML file, index.html, defines the structure and functionality of the dashboard. It includes links to styles and libraries such as jQuery, DataTables, and Chart.js to enhance interactivity and visualization.

At the top, a table is created to display weather variables, which will be populated with data through the table.js script. Below the table, various canvas elements are provided for plotting charts, which will also be filled with data via the plot.js file.

#### Javascript
   
The *table.js* script is in charge of populating with data the dashboard table. It uses the Fetch API to asynchronously retrieve data from the server at the */data* endpoint. 

The script formats numerical values for better readability and dynamically constructs table rows and cells to display the weather variables. Besides, it identifies which record is closest to the current date and time. This closest record is highlighted in the table for easy reference.

On the other hand, *plot.js* is responsible for creating dynamic visualizations using **Chart.js**. Similar to *table.js*, it fetches data from the server */data* endpoint and fills multiple bar charts, each representing a different weather parameter.

Similarly, a vertical line displaying the current date and time is created to enhance readibility and provide context.

Finally, both scripts are set to refresh periodically ensuring that the dashboard reflects the most current weather conditions available.

#### CSS
The *styles.css* file contains simple CSS to style the table and arrange the layout of the bar plots into a grid. It ensures the table is easy to read and the charts well-organized.


## Conclusion

In this project, we integrated Google Earth Engine (GEE) with a Node.js server to create a dashboard that visualizes GFS weather forecast data for Buenos Aires.

We began by retrieving, processing, and storing data using the GEE Python API, along with libraries such as XEE, Pandas, and SQLite.

Afterwards, through the Node.js server, we established a connection to serve this data via a simple API.

Finally, the dashboard, created with HTML, JavaScript, and CSS, features a dynamic table and multiple bar charts that update automatically to reflect real-time conditions.

I hope you found this project interesting and remember it’s available on my [GitHub repository](https://github.com/jm-marcenaro/GFS-Dashboard)