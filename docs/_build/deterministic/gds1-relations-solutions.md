---
interact_link: content/deterministic/gds1-relations-solutions.ipynb
kernel_name: ana
has_widgets: false
title: 'Relations and spatial joins (Problemset)'
prev_page:
  url: /deterministic/gds1-relations
  title: 'Relations and spatial joins with vector data'
next_page:
  url: /deterministic/gds2-rasters
  title: 'Working with raster data'
comment: "***PROGRAMMATICALLY GENERATED, DO NOT EDIT. SEE ORIGINAL FILES IN /content***"
---


<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
import pandas
import numpy
import geopandas
import matplotlib.pyplot as plt
import geopy
%matplotlib inline

```
</div>

</div>



# Read in the Austin 311 data
It's in the `data` folder, called `austin_311`. 



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
reports = pandas.read_csv('../data/austin_311.csv.gz')

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
reports.head()

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">



<div markdown="0" class="output output_html">
<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>request_number</th>
      <th>type_code</th>
      <th>description</th>
      <th>department</th>
      <th>method_received</th>
      <th>status</th>
      <th>location</th>
      <th>street_number</th>
      <th>street_name</th>
      <th>zipcode</th>
      <th>latitude</th>
      <th>longitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>16-00108244</td>
      <td>TRASIGMA</td>
      <td>Traffic Signal - Maintenance</td>
      <td>Transportation</td>
      <td>Phone</td>
      <td>Duplicate (closed)</td>
      <td>6001 MANCHACA RD, AUSTIN, TX 78745</td>
      <td>6001</td>
      <td>MANCHACA</td>
      <td>78745.0</td>
      <td>30.212695</td>
      <td>-97.801522</td>
    </tr>
    <tr>
      <th>1</th>
      <td>16-00108269</td>
      <td>TRASIGMA</td>
      <td>Traffic Signal - Maintenance</td>
      <td>Transportation</td>
      <td>Phone</td>
      <td>Duplicate (closed)</td>
      <td>6001 MANCHACA RD, AUSTIN, TX 78745</td>
      <td>6001</td>
      <td>MANCHACA</td>
      <td>78745.0</td>
      <td>30.212695</td>
      <td>-97.801522</td>
    </tr>
    <tr>
      <th>2</th>
      <td>16-00324071</td>
      <td>SWSDEADA</td>
      <td>ARR Dead Animal Collection</td>
      <td>Austin Resource Recovery</td>
      <td>Phone</td>
      <td>Closed</td>
      <td>2200 E OLTORF ST, AUSTIN, TX 78741</td>
      <td>2200</td>
      <td>OLTORF</td>
      <td>78741.0</td>
      <td>30.230164</td>
      <td>-97.731776</td>
    </tr>
    <tr>
      <th>3</th>
      <td>16-00108062</td>
      <td>TRASIGMA</td>
      <td>Traffic Signal - Maintenance</td>
      <td>Transportation</td>
      <td>Phone</td>
      <td>Duplicate (closed)</td>
      <td>8401 N CAPITAL OF TEXAS HWY NB, AUSTIN, TX 78759</td>
      <td>8401</td>
      <td>CAPITAL OF TEXAS</td>
      <td>78759.0</td>
      <td>30.384989</td>
      <td>-97.766471</td>
    </tr>
    <tr>
      <th>4</th>
      <td>16-00107654</td>
      <td>STREETL2</td>
      <td>Street Light Issue- Address</td>
      <td>Austin Energy Department</td>
      <td>Phone</td>
      <td>Closed</td>
      <td>300 WEST AVE, AUSTIN, TX 78703</td>
      <td>300</td>
      <td>WEST</td>
      <td>78703.0</td>
      <td>30.268090</td>
      <td>-97.751739</td>
    </tr>
  </tbody>
</table>
</div>
</div>


</div>
</div>
</div>



## What are the 10 most common types of events in the data?



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
reports.groupby('type_code').type_code.count().sort_values(ascending=False).head(10)

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">


{:.output_data_text}
```
type_code
CODECOMP    121953
ACLONAG      35776
SWSRECYC     32610
ACINFORM     30093
SWSDEADA     28561
STREETL2     24070
SWSYARDT     22930
COAACINJ     19681
HHSGRAFF     19406
WWREPORT     15192
Name: type_code, dtype: int64
```


</div>
</div>
</div>



## Fixing missing latitude/longitude values
1. how many records have a missing latitude/longitude pair? 
2. using the `location` field and the geocoding tools we discussed before, create latitude/longitude pairs for the records with missing `latitude` and `longitude` values.
3. update your dataframe with the new geocoded `longitude` and `latitude` values.
4. show that there is no missing latitude/longitude values in the updated data.



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
missing_coordinates = reports.latitude.isnull() | reports.longitude.isnull()

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
missing_coordinates.sum()

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">


{:.output_data_text}
```
5
```


</div>
</div>
</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
import geopy

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
coder = geopy.Nominatim(user_agent='scipy2019-intermediate-gds')

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
def latlng(address):
    coded = coder.geocode(address)
    return coded.latitude, coded.longitude

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
reports[missing_coordinates].location

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">


{:.output_data_text}
```
224035           10008 DORSET DR, AUSTIN, TX
259989    2400 E OLTORF ST, AUSTIN, TX 78741
340148       506 ZENNIA ST, AUSTIN, TX 78751
378492       2006 S 6TH ST, AUSTIN, TX 78704
486128    1414 WESTOVER RD, AUSTIN, TX 78703
Name: location, dtype: object
```


</div>
</div>
</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
locations = reports[missing_coordinates].location.apply(latlng).apply(pandas.Series)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
reports.loc[missing_coordinates, 'latitude'] = locations[0]
reports.loc[missing_coordinates, 'longitude'] = locations[1]

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
reports.latitude.isnull().any() | reports.longitude.isnull().any()

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">


{:.output_data_text}
```
False
```


</div>
</div>
</div>



## Fixing missing addresses



1. how many records are missing `location` field entries?
2. using the `latitude` and `longitude` fields, find the street locations for the records with missing `location` values. 
3. update your dataframe with the new `location` values. (**BONUS: Update the `street_number`, `street_name`, and `zipcode` if you can, too**)
4. show that there are no more missing `location` values in your data. 



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
missing_locations = reports.location.isnull()

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
missing_locations.sum()

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">


{:.output_data_text}
```
13
```


</div>
</div>
</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
def reverse(coordinate):
    return coder.reverse(coordinate).address

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
locations = reports.loc[missing_locations,['latitude','longitude']].apply(reverse, axis=1)
locations

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">


{:.output_data_text}
```
23659     West Gate Boulevard, Pheasant Run, Austin, Tra...
73195     11509, January Drive, Walnut Ridge, Austin, Tr...
142636    800, Gullett Street, Govalle, Austin, Travis C...
180796    6205, Shoal Creek Boulevard, Allandale, Austin...
214871    3300, Burleson Road, Parker Lane, Austin, Trav...
267054    10400, Charette Cove, Prominent Point, Jollyvi...
366275    9725, Spanish Wells Drive, Jollyville, Austin,...
421322    3902, Carmel Drive, MLK, Austin, Travis County...
441041    1528, Payton Falls Drive, Four Seasons, Austin...
476115    1615, Rutherford Lane, Berkley Square - Headwa...
491802    3930, Bee Caves Road, Ledgeway, West Lake Hill...
536128    6913, Wentworth Drive, Loma Vista, Austin, Tra...
562261    600, North Marly Way, Austin Lake Hills, Travi...
dtype: object
```


</div>
</div>
</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
reports.loc[missing_locations, ['location']] = locations

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
def maybe_get_first_number(splitstring):
    try:
        return int(splitstring[0])
    except ValueError:
        return numpy.nan

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
street_numbers = locations.str.split(',').apply(maybe_get_first_number)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
def maybe_get_streetname(splitstring):
    try:
        int(splitstring[0])
        return splitstring[1]
    except ValueError:
        return splitstring[0]

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
street_names = locations.str.split(',').apply(maybe_get_streetname)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
street_names

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">


{:.output_data_text}
```
23659        West Gate Boulevard
73195              January Drive
142636            Gullett Street
180796     Shoal Creek Boulevard
214871             Burleson Road
267054             Charette Cove
366275       Spanish Wells Drive
421322              Carmel Drive
441041        Payton Falls Drive
476115           Rutherford Lane
491802            Bee Caves Road
536128           Wentworth Drive
562261           North Marly Way
dtype: object
```


</div>
</div>
</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
zipcodes = locations.str.split(',').apply(lambda split: split[-2])

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
zipcodes

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">


{:.output_data_text}
```
23659            78748
73195            78753
142636           78702
180796           78757
214871           78741
267054           78759
366275           78717
421322           78721
441041           78754
476115     78753:78754
491802           78746
536128           78724
562261           78733
dtype: object
```


</div>
</div>
</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
reports.loc[missing_locations, ['street_number']] = street_numbers
reports.loc[missing_locations, ['street_name']] = street_names
reports.loc[missing_locations, ['zipcode']] = zipcodes

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
reports.location.isnull().any()

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">


{:.output_data_text}
```
False
```


</div>
</div>
</div>



# Check miscoded latitude, longitude values.



Use the total bounds of the austin neighborhoods data to identify observations that may be mis-coded as outside of Austin.




<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
neighborhoods = geopandas.read_file('../data/neighborhoods.gpkg')

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
known_bounds = neighborhoods.total_bounds
known_bounds

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">


{:.output_data_text}
```
array([-98.071453,  30.068439, -97.541566,  30.521356])
```


</div>
</div>
</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
too_far_ns = (reports.latitude < known_bounds[1]) | (reports.latitude > known_bounds[3])
too_far_we = (reports.longitude < known_bounds[0]) | (reports.longitude > known_bounds[2])
outside = too_far_ns | too_far_we

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
outside.sum()

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">


{:.output_data_text}
```
3365
```


</div>
</div>
</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
reports = reports[~outside]

```
</div>

</div>



## Remove duplicate reports



311 data is very dirty. Let's keep only tickets whose `status` suggests they're reports with full information that are not duplicated. 



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
reports.status.unique()

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">


{:.output_data_text}
```
array(['Duplicate (closed)', 'Closed', 'Open', 'New', 'Resolved',
       'Closed -Incomplete Information', 'Duplicate (open)',
       'Work In Progress', 'Transferred', 'TO BE DELETED', 'Pending',
       'CancelledTesting', 'Closed -Incomplete', 'Incomplete'],
      dtype=object)
```


</div>
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
reports.status.unique()

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">


{:.output_data_text}
```
array(['Closed', 'Open', 'New', 'Resolved', 'Work In Progress',
       'Transferred', 'Pending'], dtype=object)
```


</div>
</div>
</div>



## Build a GeoDataFrame from the locations



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
reports = geopandas.GeoDataFrame(reports, geometry=geopandas.points_from_xy(reports.longitude, 
                                                                            reports.latitude))

```
</div>

</div>



# Make a map of the report instances with a basemap



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
import contextily

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
reports.crs = {'init':'epsg:4326'}
reports = reports.to_crs(epsg=3857)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
basemap, basemap_extent = contextily.bounds2img(*reports.total_bounds, zoom=10, ll=False)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
plt.figure(figsize=(15,15))
plt.imshow(basemap, extent=basemap_extent)
reports.plot(ax=plt.gca(), marker='.', markersize=1, alpha=.25)

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">


{:.output_data_text}
```
<matplotlib.axes._subplots.AxesSubplot at 0x7f7f6dc6dd30>
```


</div>
</div>
<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">

{:.output_png}
![png](../images/deterministic/gds1-relations-solutions_50_1.png)

</div>
</div>
</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
reports.shape

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">


{:.output_data_text}
```
(555497, 13)
```


</div>
</div>
</div>



## How many incidents with the Public Health department are within each neighborhood?



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
reports.department.unique()

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">


{:.output_data_text}
```
array(['Austin Resource Recovery', 'Austin Energy Department',
       'Transportation', 'Animal Services Office',
       'Austin Code Department', 'Parks & Recreation Department',
       'Economic Development Department', 'Austin Water Utility',
       'Public Works', 'Health & Human Services', 'Watershed Protection',
       'Austin Water', 'Public Health',
       'Neighborhood Housing & Community Development',
       'Austin Fire Department', 'Neighborhood Housing & Community',
       'Office of Emergency Management'], dtype=object)
```


</div>
</div>
</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
health = reports.query('department == "Public Health" '
              'or department == "Health & Human Services"')

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
plt.figure(figsize=(15,15))
plt.imshow(basemap, extent=basemap_extent)
health.plot(ax=plt.gca(), marker='.', markersize=1, alpha=.25)
plt.axis(health.total_bounds[[0,2,1,3]])

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">


{:.output_data_text}
```
array([-10916755.97016249, -10858605.25412386,   3512792.57783966,
         3570235.76893318])
```


</div>
</div>
<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">

{:.output_png}
![png](../images/deterministic/gds1-relations-solutions_55_1.png)

</div>
</div>
</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
neighborhoods = neighborhoods.to_crs(epsg=3857)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
hood_counts = geopandas.sjoin(neighborhoods, health, op='contains')\
                       .groupby('hood_id').index_right.count()
neighborhoods['health_incidents'] = hood_counts.values

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
plt.figure(figsize=(15,15))
plt.imshow(basemap, extent=basemap_extent)
neighborhoods.plot('health_incidents', ax=plt.gca(), 
                   cmap='plasma', alpha=.5)
plt.axis(health.total_bounds[[0,2,1,3]])

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">


{:.output_data_text}
```
array([-10916755.97016249, -10858605.25412386,   3512792.57783966,
         3570235.76893318])
```


</div>
</div>
<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">

{:.output_png}
![png](../images/deterministic/gds1-relations-solutions_58_1.png)

</div>
</div>
</div>



# How many public health events are within 1km of each airbnb downtown?



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
listings = geopandas.read_file('../data/listings.gpkg')
listings = listings.to_crs(epsg=3857)
downtown_hoods = ('Downtown', 'East Downtown')
downtown_listings = listings.query('hood in @downtown_hoods').sort_values('id')
downtown_listings['buffer'] = downtown_listings.buffer(1000)
within_each_buffer = geopandas.sjoin(downtown_listings.set_geometry('buffer'), 
                                     health, op='contains')

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
event_counts = within_each_buffer.groupby('id').request_number.count()

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
downtown_listings['event_counts'] = event_counts.values

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
downtown_listings.sort_values('event_counts', ascending=False).head(5)[['id', 'name', 'event_counts']]

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">



<div markdown="0" class="output output_html">
<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>name</th>
      <th>event_counts</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>9950</th>
      <td>30853001</td>
      <td>Hip, Trendy Eastside Suite - 5min from Downtown</td>
      <td>1159</td>
    </tr>
    <tr>
      <th>6294</th>
      <td>20971212</td>
      <td>Awesome Eastside Rental</td>
      <td>1158</td>
    </tr>
    <tr>
      <th>9084</th>
      <td>28463789</td>
      <td>Inn Cahoots: 3 combined units, 39 beds on 6th St</td>
      <td>1135</td>
    </tr>
    <tr>
      <th>10991</th>
      <td>32833881</td>
      <td>Airy 1BR in East Austin by Sonder</td>
      <td>1130</td>
    </tr>
    <tr>
      <th>10494</th>
      <td>32203710</td>
      <td>East Austin Loft</td>
      <td>1127</td>
    </tr>
  </tbody>
</table>
</div>
</div>


</div>
</div>
</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
downtownmap, downtownmap_extent = contextily.bounds2img(*downtown_listings.buffer(1000).total_bounds, 
                                                        zoom=13, ll=False)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
plt.figure(figsize=(10,10))
plt.imshow(downtownmap, extent=downtownmap_extent)
listings.plot(color='k', marker='.', markersize=5, ax=plt.gca())
downtown_listings.plot('event_counts', ax=plt.gca())
plt.axis(downtown_listings.buffer(1000).total_bounds[[0,2,1,3]], markersize=5)
plt.title('311 Public Health incidents', fontsize=20)

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">


{:.output_data_text}
```
Text(0.5, 1.0, '311 Public Health incidents')
```


</div>
</div>
<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">

{:.output_png}
![png](../images/deterministic/gds1-relations-solutions_65_1.png)

</div>
</div>
</div>



# What's the event type that is closest to each airbnb in Austin?



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
from pysal.lib.weights.distance import get_points_array
from scipy.spatial import cKDTree

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
report_coordinates = get_points_array(reports.geometry)
airbnb_coordinates = get_points_array(listings.geometry)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
report_kdt = cKDTree(report_coordinates)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
distances, indices = report_kdt.query(airbnb_coordinates, k=2)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
listings['nearest_type'] = reports.iloc[indices[:,1]]['description'].values

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
listings.groupby('nearest_type').id.count().sort_values(ascending=False).head(20)

```
</div>

<div class="output_wrapper" markdown="1">
<div class="output_subarea" markdown="1">


{:.output_data_text}
```
nearest_type
Austin Code - Request Code Officer              2165
ARR Missed Recycling                             758
Street Light Issue- Address                      584
Animal Control - Assistance Request              562
ARR Dead Animal Collection                       523
Loose Dog                                        517
Injured / Sick Animal                            480
Water Waste Report                               336
Pothole Repair                                   282
Graffiti Abatement                               276
ARR Missed Yard Trimmings /Organics              256
ARR Brush and Bulk                               221
Traffic Signal - Dig Tess Request                218
Austin Code - Short Term Rental Complaint SR     217
Found Animal Report - Keep                       212
ARR Missed Yard Trimmings/Compost                212
Loud Commercial Music                            178
Wildlife Exposure                                173
Public Health - Graffiti Abatement               156
Animal Bite                                      150
Name: id, dtype: int64
```


</div>
</div>
</div>

