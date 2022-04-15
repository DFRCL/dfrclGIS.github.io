---
title: Multispectral indices composite
tags: Google-Earth-Engine
intro_image: "/images/images/composite-sentinel2.png"
description: Generation of RGB composite using multispectral indexes for enhaced visualization of land cover over Guatemala using Sentinel-2 MSI imagery.
intro_image_absolute: true
intro_image_hide_on_mobile: false

---

A cloudless mosaic was constructed using Sentinel-2 MSI imagery level 2A (Surface reflectance) available through Google Earth Engine API, the mosaic was subset for Guatemala country boundaries. In order to reduce the influence from shadows, cloud cover and phenologic variations throughout the year, a mean reduction was performed, giving the mean value for each pixel (for 2019-2020 period), to produce the mosaic. You can see the final mosaic here:

{% raw %}
<iframe src="https://douglasferdycl.users.earthengine.app/view/mosaicosentinel-2" width="100%" height="600px"></iframe>
{% endraw %}

Starting with previous mosaic, the computation of NDVI (Normalized Difference Vegetation Index), MNDWI (Modified Normalized Difference Water Index), and BSI (Bare Soil Index) indices was carried out. Then the resulting indices were mapped to RGB channels to construct a composite image (Red: BSI, G: NDVI, B: MNDWI ), the visible colors can be interpreted in this way:

1. Green tones are covered by vegetation with median or high photosintetic activity.
2. Red tones have high values of BSI, and low values in NDVI and MNDWI, being interpreted as urban areas. 
3. Yellow tones represent bare soil, as they have high BSI values and lower NDVI and MNDWI, but have higher NDVI than urban areas.
4. Blue tones are represented as water bodies as result in high MNDWI values. 
5. Black tones represent surfaces that have low reflectance in all bands, are caracteristic of urban related covers as asphalt. 

The resulting composite image is shown in the following interactive map:

{% raw %}
<iframe src="https://douglasferdycl.users.earthengine.app/view/clasificacionopticasar" width="100%" height="600px"></iframe>
{% endraw %}

```javascript
    Map.setCenter(-90.3674965,15.8092506,7);
    //Algorithm reference {https://custom-scripts.sentinel-hub.com/custom-scripts/sentinel-2/cby_cloud_detection/}
    
    
    /**
     * Function to mask clouds using the Sentinel-2 QA band
     * @param {ee.Image} image Sentinel-2 image
     * @return {ee.Image} cloud masked Sentinel-2 image
     */
    function maskS2clouds(image) {
      var qa = image.select('QA60');
    
      // Bits 10 and 11 are clouds and cirrus, respectively.
      var cloudBitMask = 1 << 10;
      var cirrusBitMask = 1 << 11;
    
      // Both flags should be set to zero, indicating clear conditions.
      var mask = qa.bitwiseAnd(cloudBitMask).eq(0)
          .and(qa.bitwiseAnd(cirrusBitMask).eq(0));
    
      return image.updateMask(mask).divide(10000);
    }
    
    // Map the function over one year of data and take the median.
    // Load Sentinel-2 Surface reflectance data.
    var S2 = ee.ImageCollection('COPERNICUS/S2_SR');
    
    var S2IM = S2.filterDate('2019-01-01', '2020-01-01')
                      .filterBounds(roi)
                      // Pre-filter to get less cloudy granules.
                      .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 20))
                      .map(maskS2clouds).median();
                      
    //cloud_mask
    var NDGR= S2IM.normalizedDifference(['B3','B4']);
      
    //detection threshold (B03>0.175∧NDGR>0)∨B03>0.39
    var threshold = ((S2IM.select('B3').gt(0.175)).and(S2IM.gt(0))).or(S2IM.select('B3').gt(0.39)).and(S2IM.select('B11').gt(0.1));
    
    var cloud_mask = threshold.not();
    
    var rgbVis = {
      min: 0.0,
      max: 0.3,
      bands: ['B4', 'B3', 'B2'],
    };
    
    var NDVI = S2IM.normalizedDifference(['B8','B4']);
    var MNDWI = S2IM.normalizedDifference(['B3','B8']);
    var NDBI = S2IM.normalizedDifference(['B12', 'B8']);
    var BSI = S2IM.expression("nd_2=(b('B11')+b('B4'))-(b('B8')+b('B2'))/(b('B11')+b('B4'))+(b('B8')+b('B2'))");
    
    var indices = ee.Image.cat(NDVI, MNDWI, BSI);
    
    var guatemala = departamentos.union()
    
    Map.addLayer(indices.clip(guatemala), {min:[0,0,0.1],max:[0.5,1,0.5],  bands: ['nd_2', 'nd', 'nd_1'],}, 'Clases',1);
```
