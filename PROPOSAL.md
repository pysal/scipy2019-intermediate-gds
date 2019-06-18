Intermediate Methods for Geospatial Data Analysis
=========

Keywords: geography, data science, computational geometry, modelling, statistics, visualization

*Tutorial topic: Please provide the general topic of your tutorial.*
This tutorial covers common patterns of data analysis, visualization, and modelling for geographic data. 

*__Please include a detailed description of the tutorial__. Include a detailed outline for reviewers including the duration of each part and exercise sessions. It should also include the target audience, expected level of knowledge prior to the class and the goals of the class. This description is used by the reviewers to evaluate your submission. If you decide to attach extra documentation below (under Paper), please mention it in the outline.*

This tutorial presents intermediate-level techniques for geospatial data analysis in Python. It first focuses on introducing the participants to the different libraries to work with geospatial data and will cover munging geo-data and exploring relations over space. This includes importing data in different formats (e.g. shapefile, GeoJSON), visualizing, combining and tidying them up for analysis, and will use libraries such as pandas, geopandas, shapely, PySAL, or rasterio. The second part will deal with exploratory analysis of spatial datasets, focusing on anomaly detection. No previous experience with those geospatial Python libraries is needed, but basic familiarity with geospatial data and concepts (shapefiles, vector vs raster data) and pandas will be essential.

The workshop will span half a day, and cover two core topics:

I. Working with spatial data

Spatial data is unique in a few critical ways. Its unique geographical structure requires distinct methods for data cleaning, processing, and analysis. This can make it difficult to combine different spatial datasets together, move between different geographic representations of the same observations, or leverage geographical structure inside of existing analytical workflows. This component will teach both fundamental and intermediate techniques in working with spatial data, such as:
- spatial relationships, joins, and queries
- constructing spatial data from text or addresses (geocoding)
- mixing, merging, and aligning spatial data (both imagery and vector data)

II. Exploratory spatial data analysis

Spatial data is also unique because of its special structure. Geography both embeds relationships in data--since nearby places tend to be more similar to one another than far away places--and expresses relationships in data, since maps nearly always provide a useful frame for visualizing patterns in spatial data. Fortunately, computational methods can improve upon both of these properties of spatial data to provide better analyses. This component of the workshop will focus on intermediate techniques in the analysis of spatial data, primarily focused on univariate and multivariate anomaly detection. 

*Student's Python Knowledge Level.* Please indicate the appropriate level of Python experience for attendees.*

- [ ] beginner
- [X] intermediate
- [ ] advanced

*Please include a short bio including relevant teaching experience..* If you have recorded talks or tutorials, please include the link.*
Both instructors have given workshops at numerous conferences, including SciPy in 2018 and 2016. 

Scipy 2018: https://www.youtube.com/watch?v=kJXUUO5M4ok
Scipy 2016: https://www.youtube.com/watch?v=TY4QWnnd4jY

This workshop grows out of requests and feedback from attendees about topics they would have preferred additional information on. The proposal focuses more intently on some topics in order to provide a more in-depth presentation of intermediate techniques for working with and analyzing spatial data. 
 
A comprehensive list of workshops given by the authors is provided below:
http://pysal.org/getting_started.html

*Please provide detailed set up instructions for all necessary software.c Instructions should be for various common Python environments so that attendees can have everything ready for participating before heading to SciPy.*
We recommend using the Anaconda Python distribution. A full environment will be distributed two months before the conference, but packages that will be included in the workshop are:

- geopandas
- rasterio
- scikit-learn
- pysal
- rasterstats

Most of these will be recommended from conda-forge or PyPI. We will publish full installation directions, with specific pinned versions of packages, a month and a half before the conference. 

Often, geopandas & rasterio (with their various dependencies on fiona, libgdal, and geos) are difficult for users to install and have experienced significant disruption recently in all install channels. Thus, we will also provide a conda environment to install all packages required for the workshop.

*What skills are needed to successfully participate in your tutorial? Please check all that apply:

- [ ] None
- [X] Numpy
- [ ] SciPy
- [ ] SciKit Learn
- [X] Pandas
- [ ] Matplotlib
- [ ] C++ 

*If other topics are a prerequisite, please explain further. Please note the level of mastery of the prerequisite.*

N/A

*Please provide a short summary of your topic. The summary should be less than 100 words and be suitable to be used as as description in the online program*

This tutorial will provide attendees with a tricks, tips, and techniques that are often necessary to work with geographic data in Python. These methods will range from the basics of reading & writing spatial data to the mechanics of combining and summarizing disparate geographic data types and representations. Overalln an in, participants will gain a better understanding of practical methods to bring many different geographic datasets together for analysis.
