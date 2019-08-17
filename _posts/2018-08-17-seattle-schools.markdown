---
layout: post
title:  Demographics of Seattle Schools Student Assignment Plan
date:   2017-08-25 13:32:20 +0300
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img: diversity.svg # Add image post (optional)
tags: [Blog, Geospatial]
---
In my freshman year of high school, my school district released a [new student assignment plan][assignment-plan], based on geography rather than choice, as had been the previous policy. I remember thinking that it would be interesting to study how the demographics of Seattle mapped onto the demographics of individual schools, but at the time I didn't know enough about GIS and data science to do anything about it. Well now I do, so here's an attempt.

The first task is collecting the demographic data of Seattle at the finest level of detail possible. That means collecting block-level data from the US Census Bureau. To know which Census blocks are within the Seattle city limits, we can download the shapefiles for the [Seattle 2010 Census Blocks][census-blocks-download]. The [GeoPandas][geopandas-home] library is an incredibly handy and powerful package for dealing with geospatial data, and I'll be using it extensively throughout this article.

Another library I'll be using is [census][census-package], which is a convenient wrapper for the US Census API. To use the US Census API, you need to request a [Census API Key][census-key].

Census data is structured hierarchically: states contain counties, which contain tracts, which contain block groups, which contain blocks. The Census API has limits that restrict the level of geographic detail you can fetch from a single query. For instance, you can't query all blocks in a county, but you can query all tracts in a county and then query all blocks in a given tract. Because of these rules, we first need to determine all tracts in Seattle, then query block-level data for each of those tracts.

Most shapefiles have a ``GEOID10`` field that uniquely specifies a geographic entity. The GEOID10 associated with the shapefile from the City of Seattle consists of a two digit state code, a three digit county code, a six digit tract code, and a 4 digit block code. For example, the GEOID10 "530330067001001" refers to state 53 (Washington), county 033 (King County), tract 67 (Magnolia/Queen Anne), block 1001 (SOME LOCATION). In the following code block, we extract and sort all unique tract numbers:
{% highlight python %}
import geopandas as gpd

blocks_gdf = gpd.read_file( 'path/to/2010/census/shapefile.shp' )
blocks_gdf = blocks_gdf[['GEOID10', 'geometry']]

tracts = set([k[5:11] for k in blocks_gdf['GEOID10']])
tracts = sorted(list(tracts))
{% endhighlight %}

Now we prepare the census queries. I create dicts of census variables and their corresponding descriptions, which I got from a very long list of the [2010 Census variables][census-variables]. The Census records Hispanic heritage [differently][hispanic-origin] than other ethic backgrounds, so for simplicity I'm not icluding that variable.
{% highlight python %}
CODE_DICT = {
  'P001001' : 'all',
  'P003002' : 'white',
  'P003005' : 'asian',
  'P003003' : 'black',
  'P003004' : 'native',
  'P003006' : 'pacific' }
{% endhighlight %}

Now we loop over all tracts, querying all census variables for all blocks:
{% highlight python %}
from census import Census

c = Census( "YOUR_CENSUS_KEY", year = 2010 )

tract_responses = dict()

for i, tract in enumerate(tracts):

  # get census data from te SF1 file, from 2010
  tract_responses[tract] = c.sf1.get(
    tuple(CODE_DICT.keys())),
    {
      'for' : 'block:*',
      'in': f'state:53+county:033+tract:{tract}'
    }
  )
{% endhighlight %}
The next step is some ugly code to coalesce the census data into a single pretty GeoDataFrame. You can read it on my GitHub, but I'm omitting it here for the sake of brevity. For fun, I've included some plots of population density by block for different ethnic backgrounds.

<img align="left" src="/assets/img/posts/2018-08-17-seattle-schools/white.svg" width="300">
<img align="center" src="/assets/img/posts/2018-08-17-seattle-schools/black.svg" width="300">
<img align="right" src="/assets/img/posts/2018-08-17-seattle-schools/asian.svg" width="300">

<!-- ![alt-text-1](/assets/img/posts/2018-08-17-seattle-schools/white.svg "White") ![alt-text-2](/assets/img/posts/2018-08-17-seattle-schools/asian.svg "Asian") ![alt-text-3](/assets/img/posts/2018-08-17-seattle-schools/black.svg "Black") -->


[assignment-plan]: https://www.seattleschools.org/UserFiles/Servers/Server_543/File/District/Departments/Enrollment%20Planning/Student%20Assignment%20Plan/New%20Student%20Assignment%20Plan.pdf
[census-blocks-download]: http://data-seattlecitygis.opendata.arcgis.com/datasets/38105e262d9441b59b2dde020cb02b40_13.zip
[geopandas-home]: http://geopandas.org/
[census-package]: https://github.com/datamade/census
[census-key]: https://api.census.gov/data/key_signup.html
[census-variables]: https://api.census.gov/data/2010/dec/sf1/variables.html
[hispanic-origin]: https://www.census.gov/prod/cen2010/briefs/c2010br-02.pdf