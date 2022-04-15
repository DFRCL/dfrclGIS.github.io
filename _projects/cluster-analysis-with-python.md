---
title: Cluster analysis with Python
tags: Python
intro_image: "/images/images/clusters.png"
intro_image_absolute: true
intro_image_hide_on_mobile: false
description: 'A brief example of implementation of different cluster analysis algorithms
  including: K-means, DBSCAN, HDBSCAN.'

---
This project included cluster analysis for a series of event points with geographic coordinates. It explored three algorithms: **K-means, DBSCAN, HDBSCAN.**

Initially, implementation of K-means was undertaken, for determining the optimal cluster number a silhouette score was computed, allowing for assessing the overall accuracy of the proposed K.

The results maps were generated using the Folium module in Python 3. 
{% include /maps/kmeans.html %}
{% include /maps/clusters.html %}
