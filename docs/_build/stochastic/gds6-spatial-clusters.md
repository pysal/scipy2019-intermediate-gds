---
interact_link: content/stochastic/gds6-spatial-clusters.ipynb
kernel_name: python3
has_widgets: false
title: 'Spatial clusters and regionalization'
prev_page:
  url: /stochastic/gds5-exploration-solutions
  title: 'Exploring the Trump vote (Problemset)'
next_page:
  url: 
  title: ''
comment: "***PROGRAMMATICALLY GENERATED, DO NOT EDIT. SEE ORIGINAL FILES IN /content***"
---


# Clustering and Regions

The previous notebook provided several illustrations of the power of
visualization in the analysis of spatial data. This power stems from
visualizations ability to tap into our human pattern recognition machinery.

In this notebook we introduce methods for regionalization and clustering.


## Imports



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
import pandas as pd
import geopandas as gpd
import libpysal as lp
import matplotlib.pyplot as plt
import rasterio as rio
import numpy as np
import contextily as ctx
import shapely.geometry as geom
%matplotlib inline

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
neighborhoods = gpd.read_file('../data/neighborhoods.gpkg')
# was created in previous notebook with neighborhoods.to_file('data/neighborhoods.gpkg')
listings = gpd.read_file('../data/listings.gpkg')
# was created in previous notebook with listings.to_file('data/neighborhoods.gpkg')

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
listings['price'] = listings.price.str.replace('$','').str.replace(',','_').astype(float)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
neighborhoods.head()

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
summaries = gpd.sjoin(listings[['price', 'accommodates', 'geometry']], neighborhoods, op='within')\
                .eval('price_per_head = price / accommodates')\
                .groupby('index_right')\
                .agg(dict(price_per_head='median',
                          price='median'))

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
neighborhoods['median_pph'] = summaries.price_per_head
neighborhoods['median_pri'] = summaries.price

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
f,ax = plt.subplots(1,2,figsize=(8,3))
neighborhoods.plot(column='median_pri', ax=ax[0])
neighborhoods.plot(column='median_pph', ax=ax[1])
ax[0].set_title('Median Price')
ax[1].set_title('Median Price per Head')

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
import libpysal

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
wq = libpysal.weights.Queen.from_dataframe(neighborhoods)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
from sklearn.cluster import KMeans, AgglomerativeClustering

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
kmeans = KMeans(n_clusters=5)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
import numpy
numpy.random.seed(0)
cluster_variables = ['median_pri']
k5cls = kmeans.fit(neighborhoods[cluster_variables])

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
k5cls.labels_

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
# Assign labels into a column
neighborhoods['k5cls'] = k5cls.labels_
# Setup figure and ax
f, ax = plt.subplots(1, figsize=(9, 9))
# Plot unique values choropleth including a legend and with no boundary lines
neighborhoods.plot(column='k5cls', categorical=True, legend=True, linewidth=0, ax=ax)
# Remove axis
ax.set_axis_off()
# Keep axes proportionate
plt.axis('equal')
# Add title
plt.title(r'Price Clusters (k-means, $k=5$)')
# Display the map
plt.show()

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
k5sizes = neighborhoods.groupby('k5cls').size()
k5sizes

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
k5means = neighborhoods.groupby('k5cls')[cluster_variables].mean()

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
k5means.T

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
import numpy
numpy.random.seed(0)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
ward5 = AgglomerativeClustering(linkage='ward', n_clusters=5)
ward5.fit(neighborhoods[cluster_variables])
neighborhoods['ward5'] = ward5.labels_

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
ward5sizes = neighborhoods.groupby('ward5').size()
ward5sizes

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
ward5means = neighborhoods.groupby('ward5')[cluster_variables].mean()
ward5means.T

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
# Setup figure and ax
f, ax = plt.subplots(1, figsize=(9, 9))
# Plot unique values choropleth including a legend and with no boundary lines
neighborhoods.plot(column='ward5', categorical=True, legend=True, linewidth=0, ax=ax)
# Remove axis
ax.set_axis_off()
# Keep axes proportionate
plt.axis('equal')
# Add title
plt.title('Price Clusters (AHC, $k=5$)')
# Display the map
plt.show()

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
# Setup figure and ax
f, axs = plt.subplots(1, 2, figsize=(12, 6))

ax = axs[0]
# Plot unique values choropleth including a legend and with no boundary lines
neighborhoods.plot(column='ward5', categorical=True, cmap='Set2', 
                   legend=True, linewidth=0, ax=ax)
# Remove axis
ax.set_axis_off()
# Keep axes proportionate
ax.axis('equal')
# Add title
ax.set_title('K-Means solution ($k=5$)')

ax = axs[1]
# Plot unique values choropleth including a legend and with no boundary lines
neighborhoods.plot(column='k5cls', categorical=True, cmap='Set3',
                   legend=True, linewidth=0, ax=ax)
# Remove axis
ax.set_axis_off()
# Keep axes proportionate
ax.axis('equal')
# Add title
ax.set_title('AHC solution ($k=5$)')

# Display the map
plt.show()



```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
numpy.random.seed(123456)
model = AgglomerativeClustering(linkage='ward',
                                connectivity=wq.sparse,
                                n_clusters=5)
model.fit(neighborhoods[cluster_variables])

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
neighborhoods['ward5wq'] = model.labels_
# Setup figure and ax
f, ax = plt.subplots(1, figsize=(9, 9))
# Plot unique values choropleth including a legend and with no boundary lines
neighborhoods.plot(column='ward5wq', categorical=True, legend=True, linewidth=0, ax=ax)
# Remove axis
ax.set_axis_off()
# Keep axes proportionate
plt.axis('equal')
# Add title
plt.title(r'Price Regions (Ward, $k=5$, Queen Contiguity)')
# Display the map
plt.show()

```
</div>

</div>



## Spatial Clustering Based on Points



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
listings.shape

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
clipped_thiessens = gpd.read_file('../data/thiessens.gpkg')

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
clipped_thiessens.shape

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
wtq = libpysal.weights.Queen.from_dataframe(clipped_thiessens)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
wtq.n

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
numpy.random.seed(123456)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
logprice = numpy.log(listings[['price']]+1)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
model = AgglomerativeClustering(linkage='ward',
                                            connectivity=wtq.sparse,
                                            n_clusters=5)
model.fit(logprice)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
listings['region'] = model.labels_

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
numpy.unique(model.labels_, return_counts=True)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
listings.sort_values('region').plot(column='region', marker='.')

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
region_sizes = listings.groupby('region').size()
region_sizes

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
model = AgglomerativeClustering(linkage='ward',n_clusters=5)
model.fit(logprice)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
listings['region5'] = model.labels_

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
listings.groupby('region5').size()

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
region1 = listings[listings.region5==1]

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
region1.plot()

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
for region in range(5):
    region_ = listings[listings.region5==region]
    region_.plot()
    print(region_.shape)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
region_.shape

```
</div>

</div>

