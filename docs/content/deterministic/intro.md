Intermediate Deterministic Spatial Analysis
=======================================

Spatial analysis is a broad domain. From cartographic analysis to complex statistical modeling, what “spatial analysis” means will often depend on what the “analysis” is in the first place. One of the most common kinds of spatial analysis is deterministic spatial analysis. Deterministic spatial analysis is the set of techniques, approaches, tools, and algorithms that are required to understand, describe, or explain empirical structures in geographic data. It can be divided into a few separate kinds of operations:

### Spatial queries & relationships

How many gas stations are within a mile of my broken down car? What is the closest tow truck to my location? What is the average price of a hotel in this city? Questions like these are intrinsically spatial, since they involve questions about the *spatial* relationship between features under study. Further, they tend to require specialized software and algorithms to answer. Broadly speaking, operations of this type can be split into a few common aims:

1. **Spatial Query**: Determining *what* a spatial relationship is. (*How far away is Mick’s Garage from me?*)
2. **Spatial Summary**: Determining *summaries* of entities that satisfy a spatial *predicate*, or search criteria (*How many garages are within a mile’s walk of me? What is the average price of an inspection at these garages?*)
3. **Spatial Join**: Linking entities together based on a spatial relationship (*Which major road is each garage nearest to?*)

Nearly any problem in spatial analysis can be linked to one of these three fundamental concepts. 

## What is covered

In the following chapters, we will cover questions on all three topics in both *raster* (i.e. image) and *vector* data. Raster data is a familiar representation for most; expressed as an image, each site, or pixel, has a distinct value. Vector data, on the other hand, tends to be represented as a *table*, where each record has a distinct shape. That shape may be a point, a line, a polygon, or a collection of any of those three types. For instance, a dataset representing population in the US might take a *raster* representation if it provides a population surface, giving the count of people within a 2 kilometer by 2 kilometer square. The same dataset may take on a *vector* representation if it gives the count of people within each *US Census Block*, each of which has a distinct shape. Working between the two representations is important, and we will cover that briefly in this section. 

The first notebook focuses on *vector* data. It covers spatial queries, summaries, and joins. The second notebook focuses on *raster* data, and covers mainly questions of queries and summaries. 

## What is not covered

Notably, one kind of problem in deterministic spatial analysis, that of [*spatial optimization*](<https://www.tandfonline.com/doi/full/10.1080/00045608.2012.685044>), often cannot be reduced to these three fundamental processes. Instead, spatial optimization is the formal mathematical analysis of spatial systems in order to reconfigure or add to them in such a way that a given objective is optimized. This might include selecting the lowest cost route for delivery trucks, the best location to site a warehouse, or the smallest number & locations of streetlights needed to illuminate a suburb. Spatial optimization is a deep field with strong conceptual ties to mathematics and operations research. Because of its complexity and particularity, we omit it from this workshop. 

Deterministic spatial analysis usually does *not* involve statistical estimation and inference.  But, thinking of statistics as *“a numerical summary of data,”* then simple empirical summaries are indeed be part of deterministic spatial analysis. More complicated *models* of spatial relationships are instead part of *stochastic* spatial analysis, which relates spatial structure to an underlying theory of process at hand. We will discuss this further in the *Stochastic Spatial Analysis* section of this workshop. But, questions about clustering, spatial randomness, segregation, or spatial models will be reserved for the *stochastic spatial analysis* section.