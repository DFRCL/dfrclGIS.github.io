---
title: Imágen óptica compuesta basada en índices espectrales con Sentinel-2
tags: Google Earth Engine
intro_image: "/images/images/composite-sentinel2.png"
description: Generación de imagen compuesta en bandas RGB a partir de índices multiespectrales
  con imágenes de Sentinel 2 (Reflectancia de superficie)., para mostrar una clasificación
  básica del territorio guatemalteco.
intro_image_absolute: true
intro_image_hide_on_mobile: false

---
Inicialmente se realizó un mosaico a partir de imágenes de Sentinel-2 (resolución 10 m) disponibles a través de la API de Google Earth Engine para el territorio de Guatemala. Con el objetivo de reducir la influencia de sombras, nubosidad y fenología se generó la mediana de un año de imágenes (período 2019-2020) para producir el mosaico. A continuación se presenta el mosaico.

{% raw %}
<iframe src="https://douglasferdycl.users.earthengine.app/view/mosaicosentinel-2" width="100%" height="600px"></iframe>
{% endraw %}

A partir del mosaico anterior de la misión Sentinel-2 se ejecutó un análisis consistente en varios índices espectrales: NDVI, MNDWI y BSI. La imagen generada para mostrar la clasificación es una composición RGB de los tres índices, por lo que se puede inferir lo siguiente:

1. Las áreas en color verde están cubiertas por vegetación con mediana o alta capacidad fotosintética.
2. Las áreas resaltadas en color rojo, tienen altos valores del índice BSI(Bare soil index), por lo que representan zonas urbanas.
3. Las zonas que aparecen con colores amarillos representan suelo descubierto, pero que no están cubiertos con materiales artificiales, debido a que poseen un alto valor de BSI, pero no tan bajo de NDVI.
4. Naturalmente las zonas con color azul, poseen un alto valor del MNDWI (Modified Normalized Difference Water Index).
5. Las áreas en negro son zonas que tienden a ser superficies urbanas pero con baja reflectancia, por lo que exhiben un valor bajo en la mayoría de las bandas.

{% raw %}
<iframe src="https://douglasferdycl.users.earthengine.app/view/clasificacionopticasar" width="100%" height="600px"></iframe>
{% endraw %}

```Javascript
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
