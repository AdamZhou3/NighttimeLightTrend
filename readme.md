Armed conflicts https://www.prio.org/Data/Armed-Conflict/



Data Source Nighttime lights https://developers.google.com/earth-engine/datasets/catalog/NOAA_DMSP-OLS_NIGHTTIME_LIGHTS#description



https://www.nytimes.com/interactive/2015/03/12/world/middleeast/syria-civil-war-after-four-years-map.html



Night-time lighting data can be used as a reflection of regional economic development. In this project, we compute a linear fit over the time series of values at each pixel, visualizing the y-intercept in green and positive/negative slopes as red/blue. This graph allows us to see the trends in night-time light worldwide near 30 years, from 1991 to 2014. The green areas were very bright in 1991, while the blue and red areas show the decline and rise of night-time lighting.

https://experiments.withgoogle.com/collection/chrome

```javascript
// Compute the trend of night-time lights.

// Adds a band containing image date as years since 1991 to 2014.
function createTimeBand(img) {
  var year = ee.Date(img.get('system:time_start')).get('year').subtract(1991);
  return ee.Image(year).byte().addBands(img);
}

// Map the time band creation helper over the night-time lights collection.
// https://developers.google.com/earth-engine/datasets/catalog/NOAA_DMSP-OLS_NIGHTTIME_LIGHTS
var collection = ee.ImageCollection('NOAA/DMSP-OLS/NIGHTTIME_LIGHTS')
    .select('stable_lights')
    .map(createTimeBand);
print(collection)

// Compute a linear fit over the series of values at each pixel, visualizing
// the y-intercept in green, and positive/negative slopes as red/blue.

var ls = collection.reduce(ee.Reducer.linearFit())
var light = ls.visualize({
  min: 0, 
  max: [0.20, 20, -0.18], 
  bands: ['scale', 'offset', 'scale']
});
Map.addLayer(light);

// resample to a lower resolution 
var light_84=light.reproject({crs:"EPSG:4326",scale:5000});
Map.addLayer(light_84);
var geometry = ee.Geometry.Rectangle([-180, -90, 180, 90],"EPSG:4326", false);
Map.addLayer(geometry)

//// export to google drive 
Export.image.toDrive({
  image: light_84,
  description: 'imageToDriveExample',
  fileFormat:"GeoTIFF",
  region: geometry
});
```





```python
import pandas as pd
from random import randrange
import json

wb=pd.read_excel("ConflictSite 4-2010_v3 Dataset.xls")
wb=wb[wb["Radius"]>0][['Latitude', 'Longitude', 'Radius']]

jsondata = []
for i in range(wb.shape[0]):
    raw = wb.iloc[i,:]
    jsondata.extend([raw[0],raw[1],raw[2]/2000])

with open("armedconflicts.json","w") as outfile:
    json.dump([["G", jsondata]], outfile)
    outfile.close()
```

