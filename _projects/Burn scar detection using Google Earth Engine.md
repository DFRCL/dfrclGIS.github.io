---
title: Burn scar detection using Google Earth Engine
tags: Google-Earth-Engine
intro_image: "/images/images/nbr_analysis.png"
description: Burn scar analysis with NBR index using Sentinel-2 imagery and processing
  with Google Earth Engine

---
Webapp using Google Earth Engine Javascript API, allowing to visualize burned areas due to anthropic activity or forest fires. The web app takes **Sentinel-2** (MSI) L2A imagery, then parses for given timeframes, applying cloud masks and multitemporal filtering to construct a cloudless mosaic for the area.

Then calculate the **NBR** index (Normalized Burn Ratio) and analyses the differences between the initial and final period to identify burned areas, the difference in NBR is thresholded to determine which areas represent burn scars.

The area shown corresponds to the south coast of Guatemala, in this area, the predominant land cover is sugar cane crops, which apply burning land as part of their cropping techniques, as can be noticed on the resulting map.

{% raw %}<iframe src="https://douglasferdycl.users.earthengine.app/view/superficies-potencialmente-calcinadas-guatemala" width="100%" height="500px"></iframe>{% endraw %}

The code generated for analysis is shown here:

\`\`\`javascript

/** * Function to mask clouds using the Sentinel-2 QA band * @param {ee.Image} image Sentinel-2 image * @return {ee.Image} cloud masked Sentinel-2 image */ function maskS2clouds(image) { var qa = image.select('QA60'); // Bits 10 and 11 are clouds and cirrus, respectively. var cloudBitMask = 1 << 10; var cirrusBitMask = 1 << 11; // Both flags should be set to zero, indicating clear conditions. var mask = qa.bitwiseAnd(cloudBitMask).eq(0) .and(qa.bitwiseAnd(cirrusBitMask).eq(0)); return image.updateMask(mask).divide(10000); } // Map the function over the period of study and take the median. // Load Sentinel-2 Surface reflectance data. var Inicial = ee.ImageCollection('COPERNICUS/S2_SR'); var Final= ee.ImageCollection('COPERNICUS/S2_SR'); //generate outer boundary for country var guatemala = departamentos.union(); //center map view var mapcenter = guatemala.geometry().centroid().coordinates(); Map.setCenter(ee.Number(mapcenter.get(0)).getInfo(), ee.Number(mapcenter.get(1)).getInfo(), 7); //filter Image collections by region of interest, dates and less cloudy var inicial = Inicial.filterDate('2019-01-01', '2019-01-31') .filterBounds(roi) // Pre-filter to get less cloudy granules. .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 20)) .map(maskS2clouds).median() var final = Final.filterDate('2020-01-01', '2020-01-31') .filterBounds(roi) // Pre-filter to get less cloudy granules. .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 20)) .map(maskS2clouds).median(); //enhanced cloud mask for non cirrus clouds function cloud_mask(image) { var NDGR = image.normalizedDifference(\['B3','B4'\]); //detection threshold (B03>0.175∧NDGR>0)∨B03>0.39 var threshold = ((image.select('B3').gt(0.175)).and(NDGR.gt(0))).or(image.select('B3').gt(0.39)).and(image.select('B11').gt(0.1)); var cloud_msk = threshold.not() return image.updateMask(cloud_msk) } //calculate NBR var NBR_final = cloud_mask(final).normalizedDifference(\['B8','B12'\]); var NBR_inicial = cloud_mask(inicial).normalizedDifference(\['B8','B12'\]); //calculate change in NBR var difNBR= NBR_inicial.subtract(NBR_final); //water mask var MNDWI= final.normalizedDifference(\['B3','B8'\]); var w_threshold = 0.1; var agua = MNDWI.gt(w_threshold).not(); //high NBR change mask (NBR difference > 0.27 is considered as burned surface) var threshold = 0.27; var incendios = difNBR.gt(threshold); var areas_incendios = difNBR.updateMask(agua).updateMask(incendios); //clip the resulting raster to the boundaries of Guatemala var areas_incendios_gtm = areas_incendios.clip(guatemala); //Show the final result Map.addLayer(areas_incendios_gtm,{min: 0.27, max: 0.66, palette: \['ffeb00', 'd67702', 'a37236','d63000'\]},'2020 - 2019');

\` \` \`

**Note:** If you want to replicate this code, take into account that ROI is a custom feature class defined for this analysis, you can create your own Region of Interest geometry to run the analysis.