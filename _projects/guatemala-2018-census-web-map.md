---
title: Guatemala 2018 census web map
tags: Web maps, leaflet
intro_image: "/images/images/mapa_censo.png"
description: Interactive web map of Guatemala's census data from 2018, using Leaflet.js

---
To generate this web map, census data was obtained from Guatemala National Statistics Institute (INE), then it was joined through GIS data relations to create a layer for municipalities.

The census data layer was uploaded via QGIS to a PostGIS database, and then plotted using Leaflet. Then population density was computed using existing attributes, it was applied styling based on the population density variable. It was implemented with popups to show information interactively. 

Another heatmap map was created to show population distribution in the country. The hotspot analysis was performed using QGIS and then uploaded to Mapbox to create a custom style to be displayed with Leaflet.js on website.