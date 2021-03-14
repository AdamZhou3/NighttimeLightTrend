# Nighttime Lights Trend Globe-Visualization

Nighttime lights data can be used as a reflection of regional economic prosperity/recession and poverty. In this project, we compute a linear fit over the time series of values at each pixel, visualizing the y-intercept in green and positive/negative slopes as red/blue. This graph allows us to see the trends in night-time light worldwide for nearly 30 years, from 1991 to 2014. The green areas were relatively developed in 1991, while the blue and red areas show the decline and rise of night-time lighting.

Live version at: https://adamzhou3.github.io/NighttimeLightTrend/

## Data Source 

[DMSP-OLS Nighttime lights]( https://developers.google.com/earth-engine/datasets/catalog/NOAA_DMSP-OLS_NIGHTTIME_LIGHTS#description)

##  Methods

1.  Use Google Earth Engine to load, manipulate and composite the DMSP OLS nighttime lights time series data.

```javascript
// Compute the trend of night-time lights.

// Adds a band containing image date as years since 1991.
function createTimeBand(img) {
  var year = ee.Date(img.get('system:time_start')).get('year').subtract(1991);
  return ee.Image(year).byte().addBands(img);
}

// Map the time band creation helper over the night-time lights collection.
// https://developers.google.com/earth-engine/datasets/catalog/NOAA_DMSP-OLS_NIGHTTIME_LIGHTS
var collection = ee.ImageCollection('NOAA/DMSP-OLS/NIGHTTIME_LIGHTS')
    .select('stable_lights')
    .map(createTimeBand);

// Compute a linear fit over the series of values at each pixel, visualizing
// the y-intercept in green, and positive/negative slopes as red/blue.
var light = collection.reduce(ee.Reducer.linearFit()).visualize({
  min: 0, 
  max: [0.18, 20, -0.18], 
  bands: ['scale', 'offset', 'scale']
});

// mask waters and ocean in darkgrey
// Load or import the Hansen et al. forest change dataset.
var hansenImage = ee.Image('UMD/hansen/global_forest_change_2015');
// Select the land/water mask.
var datamask = hansenImage.select('datamask');
// Create a binary mask.
var mask = datamask.eq(1);
// Make a water image out of the mask.
var water = mask.not();
// Mask water with itself to mask all the zeros (non-water).
water = water.mask(water);
// Make an image collection of visualization images.
var light_masked = ee.ImageCollection([
  light,
  water.visualize({palette: '121212'}),
]).mosaic();

// resample to lower resolution 
var light_84=light_masked.reproject({crs:"EPSG:4326",scale:5000});
Map.addLayer(light_84);

// export to google drive 
var geometry = ee.Geometry.Rectangle([-180, -90, 180, 90],"EPSG:4326", false);
Export.image.toDrive({
  image: light_84,
  description: 'imageToDriveExample',
  fileFormat:"GeoTIFF",
  scale: 5000,
  region: geometry
});
```

2.  HTML (index.html) to create a WebGL globe that visualized the data.

## Sources of inspiration

[Satellite photographs taken over Syria found that the country is 83 percent darker at night than before the war](https://www.nytimes.com/interactive/2015/03/12/world/middleeast/syria-civil-war-after-four-years-map.html)

[WebGL Globe by google](https://experiments.withgoogle.com/collection/chrome)

[Google Earth Engine API](https://github.com/jcline12/google-earthengine-api)

[RIchard Vijgen's Blog](https://www.richardvijgen.nl/#world)

Armed conflicts https://www.prio.org/Data/Armed-Conflict/

