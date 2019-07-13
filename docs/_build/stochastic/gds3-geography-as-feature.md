---
interact_link: content/stochastic/gds3-geography-as-feature.ipynb
kernel_name: python3
has_widgets: false
title: 'Feature engineering with geography'
prev_page:
  url: /stochastic/intro
  title: 'Stochastic Spatial Analysis'
next_page:
  url: /stochastic/gds4-visualization
  title: 'Visualizing geographic data using mapclassification'
comment: "***PROGRAMMATICALLY GENERATED, DO NOT EDIT. SEE ORIGINAL FILES IN /content***"
---


# Geography as Feature



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



Today, we'll talk about representing spatial relationships in Python using PySAL's *spatial weights* functionality. This provides a unified way to express the spatial relationships between observations. 

First, though, we'll need to read in our data built in the `relations.ipynb` notebook: Airbnb listings & nightly prices for neighbourhoods in Austin. 



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
listings = gpd.read_file('../data/listings.gpkg').to_crs(epsg=3857)
neighborhoods = gpd.read_file('../data/neighborhoods.gpkg').to_crs(epsg=3857)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
listings.head()

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
listings.hood

```
</div>

</div>



Further, we'll grab a basemap for our study area using `contextily`. Contextily is package designed to provide basemaps for data. It's best used for data in webmercator or raw WGS longitude-latitude coordinates.

Below, we are going to grab the basemap images for the `total_bounds` of our study area at a given zoom level. Further, we are specifying a different tile server from the default, the [Stamen Maps `toner-lite` tiles](http://maps.stamen.com/m2i/#toner-lite/1500:1000/12/47.5462/7.6196), to use since we like its aesthetics. 



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
basemap, bounds = ctx.bounds2img(*listings.total_bounds, zoom=10, 
                                 url=ctx.tile_providers.ST_TONER_LITE)

```
</div>

</div>



Spatial plotting has come a long way since we first started in spatial data science. But, a few tricks for `geopandas` are still somewhat arcane, so it's useful to know them.



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
f = plt.figure(figsize=(8,8))
ax = plt.gca()
# TRICK 1: when you only want to plot the boundaries, not the polygons themselves:
neighborhoods.boundary.plot(color='k', ax=ax)
ax.imshow(basemap, extent=bounds, interpolation='bilinear')
ax.axis(neighborhoods.total_bounds[np.asarray([0,2,1,3])])
# TRICK 2: Sorting the data before plotting it will ensure that 
#          the highest (or lowest) categories are prioritized in the plot.
#          Use this to mimick blending or control the order in which alpha blending might occur. 
listings.sort_values('price').plot('price', ax=ax, marker='o', cmap='plasma', alpha=.5)

```
</div>

</div>



# Spatial Weights: expressing spatial relationships mathematically



Spatial weights matrices are mathematical objects that are designed to express the inter-relationships between sites in a given geolocated frame of analysis. 

This means that the relationships between each site (of which there are usually $N$) to every other site is *represented* by the weights matrix, which is some $N \times N$ matrix of "weights," which are scalar numerical representations of these relationships.
In a similar fashion to *affinity matrices* in machine learning, spatial weights matrices are used in a wide variety of problems and models in quantitative geography and spatial data science to express the spatial relationships present in our data. 

In python, PySAL's `W` class is the main method by which people construct & represent spatial weights. This means that arbitary inter-site linkages can be expressed using one dictionary, and another *optional* dictionary: 

- **a `neighbors` dictionary,** which encodes a *focal observation*'s "name" and which other "named" observations the focal is linked.
- **a `weights` dictionary,** which encodes how strongly each of the neighbors are linked to the focal observation. 

Usually, these are one-to-many mappings, dictionaries keyed with the "focal" observation and values which are lists of the names to which the key is attached.

An example below shows three observations, `a`,`b`, and `c`, arranged in a straight line:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
neighbors = dict(a = ['b'],
                 b = ['a','c'],
                 c = ['b']
                 )

```
</div>

</div>



Connectivity strength is recorded in a separate dictionary whose keys should align with the `neighbors`:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
weights = dict(a = [1],
               b = [.2, .8],
               c = [.3]
                )

```
</div>

</div>



To construct the most generic spatial weights object, only the `neighbors` dictionary is required; the `weights` will assumed to be one everywhere. 



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
binary = lp.weights.W(neighbors) # assumes all weights are one

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
binary.weights

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
weighted = lp.weights.W(neighbors, weights=weights)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
weighted.weights

```
</div>

</div>



# Constructing different types of weights

By itself, this is not really useful; the hardest part of *using* these representations is constructing them from your original spatial data. Thus, we show below how this can be done. First, we cover *contiguity* weights, which are analogues to adjacency matrices . These are nearly always used for polygonal "lattice" data, but can also be used for points as well by examining their voronoi diagram. 

Second, we cover *distance* weights, which usually pertain to point data only. These tend to embed notions of distance decay, and are incredibly flexible for multiple forms of spatial data. 



# Contiguity


Contiguity weights, or "adjacency matrices," are one common representation of spatial relationships that spring to mind when modeling how polygons relate to one another. In this representation, objects are considered "near" when they touch, and "far" when they don't. adjacency is considered as a "binary" relationship, so all polygons that are near to one another are *as near as they are to any other near polygon*. 

We've got fast algos to build these kinds of relationships from `shapely`/`geopandas`, as well as directly from files (without having to read all the data in at once). 



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
Qneighbs = lp.weights.Queen.from_dataframe(neighborhoods)

```
</div>

</div>



The `pysal` library has gone under a bit of restructuring. 

The main components of the package are migrated to `libpysal`, which forms the base of a constellation of spatial data science packages. 


Given this, we you can plot the adjacency graph for the polygons we showed above as another layer in the plot. We will remove some of the view to make the view simpler to examine:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
f = plt.figure(figsize=(8,8))
ax = plt.gca()
# when you only want to plot the boundaries:
neighborhoods.boundary.plot(color='k', ax=ax, alpha=.4)
Qneighbs.plot(neighborhoods, edge_kws=dict(linewidth=1.5, color='orangered'), 
              node_kws=dict(marker='*'), ax=ax)
plt.show()

```
</div>

</div>




We can check if individual observations are disconnected using the weights object's `islands` argument:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
Qneighbs.islands

```
</div>

</div>



This is good news, as each polygon has at least one neighbor, and our graph has a single connected component.

PySAL weights can be used in other packages by converting them into their equivalent matrix representations. Sparse and dense array versions are offered, with `.sparse` providing the sparse matrix representation, and `.full()` providing the ids and dense matrix representing the graphs. 



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
spqneighbs = Qneighbs.sparse
spqneighbs.eliminate_zeros()

```
</div>

</div>



Visualizing the matrix, you can see that the adjacency matrix is very sparse indeed:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
plt.matshow(spqneighbs.toarray())

```
</div>

</div>



We can get the number of links as a percentage of all possible $N^2$ links from:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
Qneighbs.pct_nonzero

```
</div>

</div>



Which means that there are around 12.3% of all the possible connections between any two observations actually make it into the adjacency graph.



For contiguity matrices, this only has binary elements, recording 1 where two observations are linked. Everywhere else, the array is empty (zero, in a dense representation). 



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
np.unique(spqneighbs.data)

```
</div>

</div>



Fortunately for us, PySAL plays real well with scipy & other things built on top of SciPy. So, the [new compressed sparse graph (`csgraph`)](https://docs.scipy.org/doc/scipy/reference/sparse.csgraph.html) module in SciPy works wonders with the PySAL sparse weights representations. So, we often will jump back and forth between PySAL weights and scipy tools when working with these spatial representations of data. 



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
import scipy.sparse.csgraph as csgraph

```
</div>

</div>



Now, in `csgraph`, there are a ton of tools to work with graphs. For example, we could use `csgraph.connected_components`:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
number_connected, labels = csgraph.connected_components(spqneighbs)

```
</div>

</div>



And verify that we have a single connected component:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
print(number_connected, labels)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
Qconnected = lp.weights.Queen.from_dataframe(neighborhoods)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
Qconnected.plot(neighborhoods, node_kws=dict(marker='*'), edge_kws=dict(linewidth=.4))
neighborhoods.boundary.plot(color='r', ax=plt.gca())

```
</div>

</div>



In addition, we could use the `lp.w_subset` function, which would avoid re-constructing the weights again. This might help if they are truly massive, but it's often just as expensive to discover the subset as it is to construct a new weights object from this subset. 



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
Qconnected2 = lp.weights.w_subset(Qneighbs, ids=[i for i in range(Qneighbs.n) if labels[i] == 0])

```
</div>

</div>



Sometimes, if `pandas` rearranges the dataframes, these will appear to be different weights since the ordering is different. To check if two weights objects are identical, a simple test is to check the sparse matrices for **in**equality:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
(Qconnected2.sparse != Qconnected.sparse).sum()

```
</div>

</div>



### Alternative Representations

PySAL, by default, tends to focus on a single `W` object, which provides easy tools to construct & work with the accompanying sparse matrix representations. 

However, it's often the case we want alternative representations of the same relationships. 

One handy one is the weights list. This is an alternative form of expressing a weights matrix, and provides a copy of the underlying `W.sparse.data`, made more regular and put into a pandas dataframe.  



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
adjlist = Qconnected.to_adjlist()
adjlist.head()

```
</div>

</div>



This handy if you'd rather work with the representation in terms of individual edges, rather than in sets of edges. 

Also, it is exceptionally handy when you want to ask questions about the data used to generate the spatial weights, since it lets you attach this data to each of the focal pairs and ask questions about the associated data at that level. 

For example, say we get the median price of airbnbs within a given neighbourhood:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
listings.price.dtype

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
listings.price

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
price = listings[['price']].replace('[\$,]', '', regex=True).astype(float)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
price.mean(), price.max(), price.median(), price.min()

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
listings['price'] = price

```
</div>

</div>



Now, we are going to attach that back to the dataframe containing the neighbourhood information. 



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
median_prices = gpd.sjoin(listings[['price', 'geometry']], neighborhoods, op='within')\
                   .groupby('index_right').price.median()

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
median_prices.head()

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
neighborhoods = neighborhoods.merge(median_prices.to_frame('median_price'), 
                                    left_index=True, right_index=True, how='left')

```
</div>

</div>



Then, we can map this information at the neighbourhood level, computed from the individual listings within each neighbourhood:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
f = plt.figure(figsize=(8,8))
ax = plt.gca()
# when you only want to plot the boundaries:
neighborhoods.plot('median_price', cmap='plasma', alpha=.7, ax=ax)
#basemap of the area
ax.imshow(basemap, extent=bounds, interpolation='gaussian')
ax.axis(neighborhoods.total_bounds[np.asarray([0,2,1,3])])
#if you want the highest values to show on top of lower ones
plt.show()

```
</div>

</div>



Then, to examine the local relationships in price between nearby places, we could merge this information back up with the weights list and get the difference in price between every adjacent neighbourhood. 

Usually, these joins involve building links between both the focal and neighbor observation IDs. You can do this simply by piping together two merges: one that focuses on the "focal" index and one that focuses on the "neighbor" index.

Using a suffix in the later merge will give the data joined on the focal index a distinct name from that joined on the neighbor index. 



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
adjlist = adjlist.merge(neighborhoods[['hood_id', 
                                        'median_price']], 
                        left_on='focal', right_index=True, how='left')\
                  .merge(neighborhoods[['hood_id', 
                                        'median_price']], 
                         left_on='neighbor', right_index=True ,how='left', 
                         suffixes=('_focal', '_neighbor'))

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
adjlist.head()

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
adjlist.median_price_neighbor

```
</div>

</div>



Then, we can group by the `focal` index and take the difference of the prices. 



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
pricediff = adjlist[['median_price_focal', 
                     'median_price_neighbor']].diff(axis=1)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
pricediff.head()

```
</div>

</div>



We can link this back up to the original adjacency list, but first let's rename the column we want to `price_difference` and only keep that column:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
pricediff['price_difference'] = pricediff[['median_price_neighbor']]
adjlist['price_difference'] = pricediff[['price_difference']]

```
</div>

</div>



And, if we wanted to find the pair of adjacent neighbourhoods with the greatest price difference:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
adjlist.head()

```
</div>

</div>



Now, we can group by *both* the focal and neighbor name to get a meaningful list of all the neighborhood boundaries & their difference in median listing price. 



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
contrasts = adjlist.groupby(("hood_id_focal", "hood_id_neighbor"))\
                   .price_difference.median().abs()\
                   .sort_values().to_frame().reset_index()

```
</div>

</div>



For about six neighbourhood pairs (since these will be duplicate `(A,B) & (B,A)` links), the median listing price is the same:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
contrasts.query('price_difference == 0').sort_values(['hood_id_focal','hood_id_neighbor'])

```
</div>

</div>



On the other end, the 20 largest paired differences in median price between adjacent neighbourhoods is shown below:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
contrasts.sort_values(['price_difference',
                       'hood_id_focal'],
                       ascending=[False,True]).head(40)

```
</div>

</div>



## Contiguity for points



Contiguity can also make sense for point objects as well, if you think about the corresponding Voronoi Diagram and the Thiessen Polygons's adjacency graph. 

Effectively, this connects each point to a set of its nearest neighbouring points, without pre-specifying the number of points.

We can use it to define relationships between airbnb listings in our dataset. 



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
listings.sort_values('price').plot('price', cmap='plasma', alpha=.5)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
from libpysal.cg.voronoi import voronoi_frames
from libpysal.weights import Voronoi

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
lp.cg.voronoi_frames

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
lp.weights.Voronoi?

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
coordinates = np.vstack((listings.centroid.x, listings.centroid.y)).T

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
thiessens, points = voronoi_frames(coordinates)

```
</div>

</div>



However, the "natural" polygons generated by the `scipy.distance.voronoi` object may be excessively big, since some of the nearly-parallel lines in the voronoi diagram may take a long time to intersect. 



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
f,ax = plt.subplots(1,2,figsize=(2.16*4,4))
thiessens.plot(ax=ax[0], edgecolor='k')
neighborhoods.plot(ax=ax[0], color='w', edgecolor='k')
ax[0].axis(neighborhoods.total_bounds[np.asarray([0,2,1,3])])
ax[0].set_title("Where we want to work")
thiessens.plot(ax=ax[1])
neighborhoods.plot(ax=ax[1], color='w', edgecolor='k')
ax[1].set_title("The outer limit of the voronoi diagram from SciPy")
ax[0].axis('off')
ax[1].axis('off')
plt.show()

```
</div>

</div>



Fortunately, PySAL can work with this amount of observations to build weights really quickly. But, the `geopandas` overlay operation is very slow for this many polygons, so even with a spatial index, clipping these polygons to the bounding box can take a bit...



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
thiessens.shape

```
</div>

</div>



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
neighborhoods['dummy']=1

```
</div>

</div>



So, we've precomputed the clipped version of the thiessen polygons and stored them, so that we can move forward without waiting too long



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
clipper = neighborhoods.dissolve(by='dummy')

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
clipper.plot()

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
thiessens.head()

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
thiessens.crs = clipper.crs

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
clipped_thiessens = gpd.overlay(thiessens, clipper, how='intersection')

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
clipped_thiessens.head()

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
clipped_thiessens.plot()

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
clipped_thiessens.to_file('../data/thiessens.gpkg')

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



Note that, whereas the overlay operation to clean up this diagram took quite a bit of computation time if just called regularly ([and there may be plenty faster ways to do these kinds of ops](http://2018.geopython.net/#w4)), constructing the topology for all 11k Thiessen polygons is rather fast:



Just to show what this looks like, we will plot a part of one of the neighbourhoods in Austin: Hyde Park to the North of UT.



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
focal_neighborhood = 'Hyde Park'
focal = clipped_thiessens[listings.hood == focal_neighborhood]
focal = focal.reset_index()

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
focal.shape

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
focal.plot()

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
thiessen_focal_w = lp.weights.Rook.from_dataframe(focal)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
f,ax = plt.subplots(1,3,figsize=(15,5),sharex=True,sharey=True)

# plot the airbnbs across the map

listings.plot('price', cmap='plasma', ax=ax[0],zorder=0, marker='.')
# 
ax[0].set_xlim(*focal.total_bounds[np.asarray([0,2])])
ax[0].set_ylim(*focal.total_bounds[np.asarray([1,3])])
# Plot the thiessens corresponding to each listing in focal neighbourhood
listings[listings.hood == focal_neighborhood]\
        .plot('price', cmap='plasma', marker='.', ax=ax[1], zorder=0)
focal.boundary.plot(ax=ax[1], linewidth=.7)
    
thiessen_focal_w.plot(focal, node_kws=dict(marker='.',s=0), 
                      edge_kws=dict(linewidth=.5), color='b', ax=ax[2])
focal.boundary.plot(ax=ax[2], linewidth=.7)


# underlay the neighbourhood boundaries
for ax_ in ax:
    neighborhoods.boundary.plot(ax=ax_, color='grey',zorder=1)
    ax_.set_xticklabels([])
    ax_.set_yticklabels([])
ax[0].set_title("All Listings", fontsize=20)
ax[1].set_title("Voronoi for Listings in %s"%focal_neighborhood, fontsize=20)
ax[2].set_title("AdjGraph for Listings Voronoi", fontsize=20)
f.tight_layout()
plt.show()

```
</div>

</div>



# Distance



Distance weights tend to reflect relationships that work based on distance decay. Often, people think of spatial kernel functions when talking about distance weighting. But, PySAL also recognizes/uses distance-banded weights, which consider any neighbor within a given distance threshold as "near," and K-nearest neighbor weights, which consider any of the $k$-closest points to each point as "near" to that point. 

KNN weights, by default, are the only asymmetric weight PySAL will construct. However, using `csgraph`, one could prune/trim any of the contiguity or distance weights to be directed. 



### Kernel weights



These weights are one of the most commonly-used kinds of distance weights. They reflect the case where similarity/spatial proximity is assumed or expected to decay with distance.

Many of these are quite a bit more heavy to compute than the contiguity graph discussed above, since the contiguity graph structure embeds simple assumptions about how shapes relate in space that kernel functions cannot assume. 

Thus, I'll subset the data to a specific area of Austin before proceeding. 



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
listings['hood']=listings['hood'].fillna(value="None").astype(str)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
focal_listings = listings[listings.hood.str.startswith("Hyde")].reset_index()

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
focal_listings.sort_values('price').plot('price', cmap='plasma', zorder=3)
neighborhoods.boundary.plot(color='grey', ax=plt.gca())
plt.axis(focal_listings.total_bounds[np.asarray([0,2,1,3])])
plt.show()

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
Wkernel = lp.weights.Kernel.from_dataframe(focal_listings)

```
</div>

</div>



Now, if you wanted to see what these look like on the map:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
focal_listings.assign(weights=Wkernel.sparse[0,:].toarray().flatten()).plot('weights', cmap='plasma')
neighborhoods.boundary.plot(color='grey', ax=plt.gca())
plt.axis(focal_listings.total_bounds[np.asarray([0,2,1,3])])
plt.show()

```
</div>

</div>



So, clearly, near things are weighted very highly, and distant things are weighted low. 

So, if you're savvy with this, you may wonder:
> Why use PySAL kernel weights when `sklearn.pairwise.kernel_metrics` are so much faster?

Well, PySAL's got a few enhancements over and above scikit kernel functions. 
1. **pre-specified bandwidths**: using the `bandwidth=` argument, you can give a specific bandwidth value for the kernel weight. This lets you use them in optimization routines where bandwidth might need to be a parameter that's optimized by another function.
2. **fixed vs. adaptive bandwidths**: adaptive bandwidths adjust the map distanace to make things more "local" in densely-populated areas of the map and less "local" in sparsely-populated areas. This is adjusted by the...
3. **`k`-nearest neighborhood tuning**: this argument adjusts the number of nearby observations to use for the bandwidth. 

Also, many of the scikit kernel functions are also implemented. The default is the `triangular` weight, which is a linear decay with distance.

For example, an adaptive Triangular kernel and an adaptive Gaussian kernel are shown below, alongisde the same point above for comparison. 



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
Wkernel_adaptive = lp.weights.Kernel.from_dataframe(focal_listings, k=20, fixed=False)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
Wkernel_adaptive_gaussian = lp.weights.Kernel.from_dataframe(focal_listings, k=10, fixed=False, function='gaussian')

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
f,ax = plt.subplots(1,3,figsize=(12,4))
focal_listings.assign(weights=Wkernel.sparse[0,:].toarray().flatten()).plot('weights', cmap='plasma',ax=ax[0])
focal_listings.assign(weights=Wkernel_adaptive.sparse[0,:].toarray().flatten()).plot('weights', cmap='plasma',ax=ax[1])
focal_listings.assign(weights=Wkernel_adaptive_gaussian.sparse[0,:].toarray().flatten()).plot('weights', cmap='plasma',ax=ax[2])
for i in range(3):
    neighborhoods.boundary.plot(color='grey', ax=ax[i])
    ax[i].axis(focal_listings.total_bounds[np.asarray([0,2,1,3])])
    ax[i].set_xticklabels([])
    ax[i].set_yticklabels([])
ax[0].set_title("Defaults (Triangular fixed kernel, k=2)")
ax[1].set_title("Adaptive Triangular Kernel, k=20")
ax[2].set_title("Adaptive Gaussian Kernel, k=10")
f.tight_layout()
plt.show()

```
</div>

</div>



In the adaptive kernels, you also obtain a distinct bandwidth at each site:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
Wkernel_adaptive.bandwidth[0:5]

```
</div>

</div>



These are useful in their own right, since they communicate information about the structure of the density of points in the analysis frame:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
f,ax = plt.subplots(1,2,figsize=(8,4))
focal_listings.assign(bandwidths=Wkernel_adaptive.bandwidth).plot('bandwidths', cmap='plasma',ax=ax[0])
focal_listings.assign(bandwidths=Wkernel_adaptive_gaussian.bandwidth).plot('bandwidths', cmap='plasma',ax=ax[1])
for i in range(2):
    neighborhoods.boundary.plot(color='grey', ax=ax[i])
    ax[i].axis(focal_listings.total_bounds[np.asarray([0,2,1,3])])
    ax[i].set_xticklabels([])
    ax[i].set_yticklabels([])
ax[0].set_title("Adaptive Triangular Kernel, k=20")
ax[0].set_ylabel("Site-specific bandwidths", fontsize=16)
ax[1].set_title("Adaptive Gaussian Kernel, k=10")
f.tight_layout()
plt.show()

```
</div>

</div>



Areas with large adaptive kernel bandwidths are considered in "sparse" regions and areas with small adaptive bandwidths are in "dense" regions; a similar kind of logic is used by clustering algortihms descended from DBSCAN. 



### Distance bands



Conceptually, this is a binary kernel weight. All observations that are within a given distance from one another are considered "neighbors," and all that are further than this distance are "not neighbors." 

In order for this weighting structure to connect all observations, it's useful to set this to the largest distance connecting on observation to its nearest neighbor. This observation is the "most remote" observation and have at least one neighbor; every other observation is thus guaranteed to have at least this many neighbors. 

To get this "m distance to the first nearest neighbor," you can use the PySAL `min_threshold_distance` function, which requires an array of points to find the minimum distance at which all observations are connected to at least one other observation:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
point_array = np.vstack(focal_listings.geometry.apply(lambda p: np.hstack(p.xy)))
minthresh = lp.weights.min_threshold_distance(point_array)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
print(minthresh)

```
</div>

</div>



This means that the most remote observation is just over 171 meters away from its nearest airbnb. Building a graph from this minimum distance, then, is done by passing this to the weights constructor:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
dbandW = lp.weights.DistanceBand.from_dataframe(focal_listings, threshold=minthresh)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
neighborhoods.boundary.plot(color='grey')
dbandW.plot(focal_listings, ax=plt.gca(), edge_kws=dict(color='r'), node_kws=dict(zorder=10))
plt.axis(focal_listings.total_bounds[np.asarray([0,2,1,3])])
plt.show()

```
</div>

</div>



This model of spatial relationships will guarantee that each observation has at least one neighbor, and will prevent any disconnected subgraphs from existing. 



### KNNW



$K$-nearest neighbor weights are constructed by considering the nearest $k$ points to each observation as neighboring that observation. This is a common way of conceptualizing observations' neighbourhoods in machine learning applications, and it is also common in geographic data science applications. 



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
KNNW = lp.weights.KNN.from_dataframe(focal_listings, k=10)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
neighborhoods.boundary.plot(color='grey')
KNNW.plot(focal_listings,ax=plt.gca(), edge_kws=dict(color='r'), node_kws=dict(zorder=10))
plt.axis(focal_listings.total_bounds[np.asarray([0,2,1,3])])
plt.show()

```
</div>

</div>



One exceedingly-common method of analysis using KNN weights is by changing `k` repeatedly and finding better values. Thus, the KNN-weights method provides a specific method to do this in a way that avoids re-constructing its core data structure, the `kdtree`. 

Further, this can add additional data to the weights object as well. 

By default, this operates in place, but can also provide a copy of the datastructure if `inplace=False`. 



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
KNNW20 = KNNW.reweight(k=20, inplace=False)

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
neighborhoods.boundary.plot(color='grey')
KNNW20.plot(focal_listings,ax=plt.gca(), edge_kws=dict(color='r'), node_kws=dict(zorder=10))
plt.axis(focal_listings.total_bounds[np.asarray([0,2,1,3])])
plt.show()

```
</div>

</div>



Further, since KNN weights are asymmetric, special methods are provided to make them symmetric:



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
KNNW20sym = KNNW20.symmetrize()

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
(KNNW20sym.sparse != KNNW20sym.sparse.T).sum()

```
</div>

</div>



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
(KNNW20.sparse != KNNW20.sparse.T).sum()

```
</div>

</div>



In fact, these symmetrizing methods exist for any other weights type too, so if you've got an arbitrarily-computed weights matrix, it can be used in that case. 



### KNN on Polygons



While K-nearest neighbors weighting methods often make more sense for data in point formats, it's also applicable to data in polygons, were a *representative point* for each polygon is used to construct K-nearest neighbors, instead of the polygons as a whole. 


For comparison, I'll show this alongside of the Queen weights shown above for neighbourhoods in Berlin. 

When the number of nearest neighbours is relatively large compared to the usual cardinality in an adjacency graph, this results in some neighbourhoods being connected to one another more than a single-neigbourhood deep. That is, neighbourhoods are considered spatially connected even if they don't touch, since their *representative points* are so close to one another relative to the nearest alternatives. 



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
KNN_neighborhoods = lp.weights.KNN.from_dataframe(neighborhoods, k=10).symmetrize()

f,ax = plt.subplots(1,2,figsize=(8,4))
for i in range(2):
    neighborhoods.boundary.plot(color='grey',ax=ax[i])
    ax[i].set_xticklabels([])
    ax[i].set_yticklabels([])
KNN_neighborhoods.plot(neighborhoods, ax=ax[0], node_kws=dict(s=0), color='orangered')
Qconnected.plot(neighborhoods, ax=ax[1], node_kws=dict(s=0), color='skyblue')
ax[0].set_title("KNN(10)", fontsize=16)
ax[1].set_title("Queen Contiguity", fontsize=16)
f.tight_layout()
plt.show()

```
</div>

</div>



In conrast, very sparse K-nearest neighbours graphs will result in significantly different connectivity structure than the contiguity graph, since the relative position of large areas' *representative points* matters significantly for which observations it touches will be considered "connected." Further, this often reduces the density of areas in the map with small elementary units, where cardinality is often higher. 



<div markdown="1" class="cell code_cell">
<div class="input_area" markdown="1">
```python
KNN_neighborhoods = lp.weights.KNN.from_dataframe(neighborhoods, k=2).symmetrize()

f,ax = plt.subplots(1,2,figsize=(8,4))
for i in range(2):
    neighborhoods.boundary.plot(color='grey',ax=ax[i])
    ax[i].set_xticklabels([])
    ax[i].set_yticklabels([])
KNN_neighborhoods.plot(neighborhoods, ax=ax[0], node_kws=dict(s=0), color='orangered')
Qconnected.plot(neighborhoods, ax=ax[1], node_kws=dict(s=0), color='skyblue')
ax[0].set_title("KNN(2)", fontsize=16)
ax[1].set_title("Queen Contiguity", fontsize=16)
f.tight_layout()
plt.show()

```
</div>

</div>



## More representations

There are similarly more representations available and currently under development, such as a networkx interface in `W.to_networkx/W.from_networkx`. Further, we're always willing to add additional constructors or methods to provide new and interesting ways to represent geographic relationships. 

