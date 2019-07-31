# Intermediate Stochastic Spatial Analysis

Stochastic spatial analysis is much more difficult to summarize than deterministic spatial analysis. In part, this is because stochastic spatial analysis deals with any statistical analysis of a geographical process. This is as diverse as modeling and predicting:
  - movements of animals across habitats or humans within cities
  - proliferation of industrial practices within a sector
  - conditions in future climates or scenarios of climate change
  - outcomes of elections
  - seregation within cities
  - migration across countries
  - spread of opinions about a public issue
Given the proliferation of geographical information about nearly any social, phyisical, or behaviorial process we can study, nearly any problem can be considered a *geographical* problem. As such, nearly statistical analysis can be made "spatial" through the inclusion of information about spatial relationships.

Generally, this is abstracted into a few topics for stochastic spatial analysis:
- **exploratory spatial data analysis**: identifying spatial structures, such as clusters, outliers, contours, or hulls/regions.
- **point pattern analysis**: examining the spatial structure of "unmarked" point patterns, where only location is relevant, or of "marked" patterns, where the relationship between points' features and their locations is of interest. 
- **movement dynamics**: modeling "traces" (sequences of positions) or polygons representing an area over time. 
- **spatial regression modeling**: modifications of typical regression-modelling frameworks that explicitly account for spatial effects *(discussed below)*.
- **geostatistics/surface modeling**: models (typically, regression-form) that assume that observations are sampled from a smooth, continuous field, whose structure is modeled. 
- **distribution dynamics**: the examination of the spatial structure of statistical distributions over time. 

Broadly speaking, all of these approaches involve models or methods that take into account three heuristics for simplifying and representing the complexity of geographical processes:

### The first law
> Near things are more related than distant things, with some relative notion of "near"

This idea, known as "Tobler's Law" (after geographer [Waldo Tolber](https://en.wikipedia.org/wiki/Waldo_R._Tobler)) or ["the First Law of Geography"](https://en.wikipedia.org/wiki/Tobler%27s_first_law_of_geography). This idea is fundamentally driven by the concept of *distance decay*, that the interaction between two entities will tend to decay with distance. In geography, nearly everything is either assumed or believed to decay with distance; the extent to which this is true is then statistically-verified.  Classical statistical models often assume observations are independent of one another. If Tobler's law holds, this is unlikely, since nearby things will be more related to one another than distant things. 

This is usually modelled by a spatial effect that accounts for **spatial dependence**: tendency for observations, their locations, or their features to depend on one another. Empirical questions for these approaches might include:
   1. **Clustering:** Are there areas where observations tend to concentrate?
   2. **Colocation**: Do observations of the same type tend to locate nearby, or do they disperse?
   3. **Segregation**: Do observations of different classes or traits tend to be located near one another? 
   4. **Autocorrelation**: Do *similar* observations tend to locate nearby, or do they disperse? 
   5. **Autoregression**: Does a change in a "focal" site affect outcomes in nearby sites? 

### Nothing stands still
> Behaviors and processes may not be the same everywhere

This idea, often known as "spatial nonstationarity," reflects the fact that the structure or behavior of a process can change depending on where the process is realized or what's around a realization. This idea is fundamentally driven by the concept of *context dependence*. A geographical analogue to *path dependence*, when the behavior of a system depends on the past experiences of the system, the idea of *context dependence* suggests that parts of our models may not necessarily be the same everywhere. Questions here might include:
   1. **Mean Heterogeneity**: Does the average level of the process under study stay the same across the map, or is everything greener on the other side of the bridge? 
   2. **Process Heterogeneity**: Does the relationship between a predictor and the response variable stay the same across the map, or do some factors matter more to the model in some areas than in other areas?
   3. **Spatial Heteroskedasticity**: Does the variability in the process under study stay the same across the map, or are there some areas that are much more noisy than other areas?

### Any shape you like
> The "nature" of a place depends on how that place is bounded, which is itself uncertain. 

This idea, often known as the "modifiable unit area problem," reflects the fact that estimates about a "region" will depend on how that region is constructed. This is related to the [*ecological fallacy*](https://en.wikipedia.org/wiki/Ecological_fallacy) and the [*atomistic fallacy*](https://jech.bmj.com/content/56/8/588), since these fallacies incorrectly reason that things that hold for one level of analysis ought hold for another; if this were true, aggregations would not fundamentally change our conclusions. As a spatial version of [Simpson's paradox](https://en.wikipedia.org/wiki/Simpson%27s_paradox), this can be particularly pernicious.

![groups matter](https://en.wikipedia.org/wiki/Simpson%27s_paradox#/media/File:Simpsons_paradox_-_animation.gif) 

Questions in this domain might include:
1. **Regionalization**: what areas are latent in the data? Where do they start and stop?
2. **Boundary detection**: where are there clear discrepancies between two areas? 
3. **Scale-independent modeling**: how can geographical processes be modeled in a way that avoids Simpson's Paradox? 

## What is covered

Because of the complexity and diversity of stochastic spatial analysis, we cannot cover all of these topics. Indeed, no single text can. However, we do provide a thorough coverage of topics about distance decay, focusing explicitly on statistical tests of distance decay and autocorrelation, as well as questions of regionalization and boundary detection. 