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

Initially, implementation of K-means was undertaken, for determining the optimal cluster number a silhouette score was computed, allowing for assessing the overall accuracy of the proposed K. The results maps were generated using the Folium module in Python 3 and plots using matplotlib. 

{% raw %}
<script src="https://gist.github.com/DFRCL/45ccfb03601785a448c98d8ec25095e8.js"></script>
{% endraw %}

In the following map the result of K-means clustering with 70 clusters can be seen: 

{% raw %}
<iframe src="https://douglascl.xyz/assets/maps/kmeans.html" width="100%" height="600px"></iframe>
{% endraw %}

Then the same data was clustered using DBSCAN, an algorithm that takes into account the density of points to create a segmentation:

{% raw %}
<script src="https://gist.github.com/DFRCL/325899d3eb74edc52311469ba5465a91.js"></script>
{% endraw %}

Finally the result can be seen in the map:

{% raw %}
<iframe src="https://douglascl.xyz/assets/maps/dbscan.html" width="100%" height="600px"></iframe>
{% endraw %}

The last analysis was performed using HDBSCAN and giving a treatment to outliers to define more compact clusters, the code used was the following:

{% raw %}
<script src="https://gist.github.com/DFRCL/8cf3aa4f19b0a6a10a1e8508c82b7ba5.js"></script>
{% endraw %}

And the final result:

{% raw %}
<iframe src="https://douglascl.xyz/assets/maps/hdbscan.html" width="100%" height="600px"></iframe>
{% endraw %}

From the analysis of the three algorithms can be seen that HDBSCAN outperforms, allowing to take into account the density but being less vulnerable to outliers as DBSCAN. Another great advantage is that the algorithm determines the optimal number of clusters to get better accuracy unlike K-means, that need the expected number of clusters as input variable.
