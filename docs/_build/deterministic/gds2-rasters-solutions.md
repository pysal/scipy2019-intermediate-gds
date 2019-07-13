---
interact_link: content/deterministic/gds2-rasters-solutions.ipynb
kernel_name: ana
has_widgets: false
title: 'Working with raster data (Problemset)'
prev_page:
  url: /deterministic/gds2-rasters
  title: 'Working with raster data'
next_page:
  url: /stochastic/intro
  title: 'Stochastic Spatial Analysis'
comment: "***PROGRAMMATICALLY GENERATED, DO NOT EDIT. SEE ORIGINAL FILES IN /content***"
---


<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
import rasterio
import numpy
import pandas
import geopandas
import matplotlib.pyplot as plt
%matplotlib inline

```
</div>

</div>



## Read in the `austinlights.tif` file we made in the `rasters` notebook



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
nightlight_file = rasterio.open('../data/austinlights.tif')
nightlights = nightlight_file.read(1)


```
</div>

</div>



## Make a `nightlights_extent` array containing the extent of the nightlight raster.

the `bounds` attribute from the `nightlight_file` will be helpful. 



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
nightlight_extent = numpy.asarray(nightlight_file.bounds)[[0,2,1,3]]

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
plt.imshow(nightlights, cmap='hot', extent=nightlight_extent)

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">


{:.output_data_text}
```
<matplotlib.image.AxesImage at 0x7f68882bd710>
```


</div>
</div>
<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">

{:.output_png}
![png](../images/deterministic/gds2-rasters-solutions_5_1.png)

</div>
</div>
</div>



## Read in the Ausin 311 data



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
reports = pandas.read_csv('../data/austin_311.csv.gz')

```
</div>

</div>



## Clean the 311 data:

1. This time, we'll simply ignore the missing points. Drop the values with missing `latitude`, `longitude`, or `location` attributes. 
2. Keep only tickets whose `status` suggests they're reports with full information that are not duplicated. 
3. Remove impossible latitude/longitudes using the `nightlight_extent` you made earlier. 



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
reports = reports.dropna(subset=['longitude', 'latitude', 'location'])

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
to_drop = ('Duplicate (closed)', 'Closed -Incomplete Information', 'Duplicate (open)', 
           'TO BE DELETED', 'CancelledTesting', 'Closed -Incomplete', 'Incomplete')
reports = reports.query('status not in @to_drop')

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
too_far_ns = (reports.latitude < nightlight_extent[2]) | (reports.latitude > nightlight_extent[3])
too_far_we = (reports.longitude < nightlight_extent[0]) | (reports.longitude > nightlight_extent[1])
outside = too_far_ns | too_far_we

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
reports = reports[~outside]

```
</div>

</div>



## Make a geodataframe from the reports data



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
reports = geopandas.GeoDataFrame(reports, 
                                 geometry=geopandas.points_from_xy(reports.longitude, 
                                                                   reports.latitude))

```
</div>

</div>



## Identify whether your zipcodes are accurately coded
1. First, build a convex hull around points within the same area
2. Then, plot the convex hulls to see if they overlap/make sense. 



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
zipcodes = reports.groupby('zipcode').geometry\
                  .apply(lambda x: x.unary_union.convex_hull)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
zipcodes = geopandas.GeoDataFrame(zipcodes)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
zipcodes.plot(ax=plt.gca(), facecolor='none', edgecolor='k')

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">


{:.output_data_text}
```
<matplotlib.axes._subplots.AxesSubplot at 0x7f686cf91a20>
```


</div>
</div>
<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">

{:.output_png}
![png](../images/deterministic/gds2-rasters-solutions_18_1.png)

</div>
</div>
</div>



## What is the highest, lowest, and median brightness within these three zipcodes?
1. 78705
2. 78702
3. 78701



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
plt.figure(figsize=(10,10))
zipcodes.loc[[78705, 78702, 78701]].plot(ax=plt.gca(), facecolor='none', edgecolor='k')

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">


{:.output_data_text}
```
<matplotlib.axes._subplots.AxesSubplot at 0x7f686ff81ac8>
```


</div>
</div>
<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">

{:.output_png}
![png](../images/deterministic/gds2-rasters-solutions_20_1.png)

</div>
</div>
</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
targets = zipcodes.loc[[78705, 78702, 78701]]

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
from rasterio.mask import mask

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
def summarize_mask(geom, dataset=nightlight_file, **mask_kw):
    mask_kw.setdefault('crop', True)
    mask_kw.setdefault('filled', False)
    masked = mask(dataset=dataset, shapes=(geom,), **mask_kw)[0]
    return (masked.min(), numpy.ma.median(masked), masked.max())

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
targets.geometry.apply(summarize_mask)

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">


{:.output_data_text}
```
zipcode
78705    (63, 63.0, 63)
78702    (62, 63.0, 63)
78701    (56, 63.0, 63)
Name: geometry, dtype: object
```


</div>
</div>
</div>



## What is the brightness at each of the streetlight issues in the 311 dataset?




<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
streetlight_issues = reports.description.apply(lambda string: ('street light' in string.lower()))

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
streetlight_reports = reports[streetlight_issues]

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
brightnesses = nightlight_file.sample(streetlight_reports[['longitude', 'latitude']].values)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
brightnesses = numpy.hstack(list(brightnesses))

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
plt.figure(figsize=(10,10))
streetlight_reports.assign(brightness=brightnesses).plot('brightness', ax=plt.gca(), marker='.')

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">


{:.output_data_text}
```
<matplotlib.axes._subplots.AxesSubplot at 0x7f686e700978>
```


</div>
</div>
<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">

{:.output_png}
![png](../images/deterministic/gds2-rasters-solutions_30_1.png)

</div>
</div>
</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
plt.hist(brightnesses, bins=30, density=True)
plt.hist(nightlights.flatten(), density=True,
         bins=30, histtype='step', linewidth=2)

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">


{:.output_data_text}
```
(array([0.00083984, 0.        , 0.00503905, 0.01455726, 0.01581703,
        0.02295568, 0.01651689, 0.01623695, 0.01651689, 0.01819658,
        0.02225582, 0.01189776, 0.00853839, 0.01021808, 0.00769855,
        0.00755858, 0.0079785 , 0.00741861, 0.00727863, 0.00643879,
        0.01203774, 0.00853839, 0.00867837, 0.00783853, 0.01119789,
        0.01203774, 0.01497718, 0.02071611, 0.02827468, 0.12793595]),
 array([ 0. ,  2.1,  4.2,  6.3,  8.4, 10.5, 12.6, 14.7, 16.8, 18.9, 21. ,
        23.1, 25.2, 27.3, 29.4, 31.5, 33.6, 35.7, 37.8, 39.9, 42. , 44.1,
        46.2, 48.3, 50.4, 52.5, 54.6, 56.7, 58.8, 60.9, 63. ]),
 <a list of 1 Patch objects>)
```


</div>
</div>
<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">

{:.output_png}
![png](../images/deterministic/gds2-rasters-solutions_31_1.png)

</div>
</div>
</div>

