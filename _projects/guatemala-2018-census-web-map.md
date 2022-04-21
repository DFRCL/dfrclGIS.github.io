---
title: Guatemala 2018 census web map
tags: Web-Maps Leaflet Cartography
intro_image: "/images/censo2018.svg"
intro_image_absolute: false
intro_image_hide_on_mobile: false
description: Interactive web map of Guatemala's census data from 2018, using Leaflet.js

---
To generate this web map, census data was obtained from Guatemala National Statistics Institute (INE), then it was joined through GIS data relations to create a layer for municipalities.

The census data layer was uploaded via QGIS to a PostGIS database, and then plotted using Leaflet. Then population density was computed using existing attributes, it was applied styling based on the population density variable. It was implemented with popups to show information interactively.

{% raw %}

<iframe src="https://douglascl.xyz/assets/maps/censo2018.html" width="100%" height="600px"></iframe> {% endraw %}

The code used for generating the web map is the following:

```javascript

    <head>
       <link rel="stylesheet" href="https://unpkg.com/leaflet@1.6.0/dist/leaflet.css"
       integrity="sha512-xwE/Az9zrjBIphAcBb3F6JVqxf46+CDLwfLMHloNu6KEQCAWi6HcDUbeOfBIptF7tcCzusKFjFw2yuvEpDL9wQ=="
       crossorigin=""/>
    <script src="https://unpkg.com/leaflet@1.6.0/dist/leaflet.js"
       integrity="sha512-gZwIG9x3wUXg2hdXF6+rVkLF/0Vi9U8D2Ntg4Ga5I5BZpVkVxlJWbSQtXPSiUTtC0TjtGOmxa1AJPuV0CPthew=="
       crossorigin=""></script>
       <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.5.1/jquery.min.js"></script>
    <style>#map { width: 800px; height: 500px; }
    .info { padding: 6px 8px; font: 10px/12px Arial, Helvetica, sans-serif; background: white; background: rgba(255,255,255,0.8); box-shadow: 0 0 15px rgba(0,0,0,0.2); border-radius: 5px; } .info h4 { margin: 0 0 5px; color: #777; }
    .legend { text-align: left; line-height: 20px; color: #555; } .legend i { width: 60px; height: 18px; float: left; margin-right: 8px; opacity: 1; }</style>
    </head>
    
    <div id="map1" style="width: 100%; height: 500px"></div>
    
    <script>
    var mymap = L.map('map1').setView([15.8760489,-89.9221243], 7);
    
    var owsrootUrl = 'https://geoserverdouglascl.azurewebsites.net/geoserver/ows';
    
    var defaultParameters = {
        service : 'WFS',
        version : '1.0.0',
        request : 'GetFeature',
        typeName : 'PortafolioGIS:Poblacion_por_municipio',
        outputFormat : 'text/javascript',
        format_options : 'callback:getJson',
        SrsName : 'EPSG:4326'
    };
    
    var parameters = L.Util.extend(defaultParameters);
    var URL = owsrootUrl + L.Util.getParamString(parameters);
    
    function getColor(d) {
        return d > 2205  ? '#fdfa18' :
               d > 1425  ? '#8fd744' :
               d > 847   ? '#35b779' :
               d > 586   ? '#20908d' :
               d > 338   ? '#31688e' :
               d > 166   ? '#443A83' :
               d > 0     ? '#440154' :
                          '#FFFFFF';
    }
    
    function style(feature) {
        return {
            fillColor: getColor(feature.properties.poblacion/feature.properties.shape_area*1000000),
            weight: 2,
            opacity: 1,
            color: 'white',
            dashArray: '3',
            fillOpacity: 0.8
        };
    }
    
    function onEachFeature(feature, layer) {
                    popupOptions = {maxWidth: 300};
                    layer.bindPopup("Municipio: "+feature.properties.municipio+"</b><br> Población: "+feature.properties.poblacion+" Hab.",popupOptions);
                }
    
    var WFSLayer = null;
    var ajax = $.ajax({
        url : URL,
        dataType : 'jsonp',
        jsonpCallback : 'getJson',
        success : function (response) {
            WFSLayer = L.geoJson(response, {
    	    maxZoom: 10,
    	    attribution: '| Douglas Castillo, '+'<a href="https://www.censopoblacion.gt/">INE Guatemala</a> ',
                style: style,
                onEachFeature: onEachFeature
            }).addTo(mymap);
        }
    });
    
    L.tileLayer('https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token='MAPBOX_TOKEN', {crs: L.CRS.EPSG3857,
    		maxZoom: 18,
    		attribution: 'Map data © <a href="https://www.openstreetmap.org/">OpenStreetMap</a> contributors, ' +
    			'<a href="https://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>, ' +
    			'Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
    		id: 'mapbox/streets-v11',
    		tileSize: 512,
    		zoomOffset: -1
    	}).addTo(mymap);
    
    var legend = L.control({position: 'bottomright'});
    
    legend.onAdd = function (map) {
    
        var div = L.DomUtil.create('div', 'info legend'),
            grades = [0, 166, 338, 586, 847,1425,2205],
            labels = [];
    
        // loop through our density intervals and generate a label with a colored square for each interval
        for (var i = 0; i < grades.length; i++) {
            div.innerHTML +=
                '<i style="background:' + getColor(grades[i] + 1) + '"></i> ' +
                grades[i] + (grades[i + 1] ? '–' + grades[i + 1] + ' Hab/km²<br>' : '+');
        }
    
        return div;
    };
    
    legend.addTo(mymap);
    L.control.scale().addTo(mymap);
    
    </script>
```

Another heatmap map was created to show population distribution in the country. The hotspot analysis was performed using QGIS and then uploaded to Mapbox to create a custom style to be displayed with Leaflet.js on website.
{% raw %}

<iframe src="https://douglascl.xyz/assets/maps/population-heatmap.html" width="100%" height="600px"></iframe> {% endraw %}

