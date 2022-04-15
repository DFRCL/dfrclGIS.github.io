---
title: ''
tags: ''
intro_image: ''
description: ''
published: false

---
Webapp using Google Earth Engine Javascript API, allowing to visualize burned areas due to anthropic activity or forest fires. The web app takes **Sentinel-2** (MSI) L2A imagery, then parses for given timeframes, applying cloud masks and multitemporal filtering to construct a cloudless mosaic for the area. 

Then calculate the **NBR** index (Normalized Burn Ratio) and analyses the differences between the initial and final period to identify burned areas, the difference in NBR is thresholded to determine which areas represent burn scars. 

The area shown corresponds to the south coast of Guatemala, in this area, the predominant land cover is sugar cane crops, which apply burning land as part of their cropping techniques, as can be noticed on the resulting map.