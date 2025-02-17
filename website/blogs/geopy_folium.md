# GeoPy and Folium

## A Simple and Effective Pair

**Posted by Joe Blankenship on May 11, 2017**

In my last blog post, I created an interactive geovisualization of member thoughts on challenges facing the data science community. I did this using the member data from the meetup group I organized until recently. In order to do this, the data had to be geocoded before I could create the geovisualization. To do this I used two, very impressive Python libraries: GeoPy and Folium. I will briefly go over what each library is and how I used them to create the visualization. Then, using the code example, you can try it yourself!

## GeoPy

GeoPy is a geocoding library that taps into mapping APIs such as Google, OpenStreetMap, and many others. To ensure that your requests do not time-out, you will need API keys to uses the services. You can find out more [here](https://geopy.readthedocs.io/ "https://geopy.readthedocs.io/").

## Folium

Folium is a great way to take your data and visualize it across multiple platforms using [leaflet.js](http://leafletjs.com/ "http://leafletjs.com/"). Folium uses OpenStreetMap tiles, amongst others, as the basemap upon which your data is plotted in points, lines, polygons, or any combination of the three. You can find out more [here](https://folium.readthedocs.io/en/latest/ "https://folium.readthedocs.io/en/latest/").

## Putting It Together

First, you want to import and clean your data. Luckily, for the purpose of this data visualization, we won't need too much data from the overall member data set. The data set does come nicely structured in a TSV format.

> Note: I am using Jupyter Notebook to perform these tasks.

```python

# Import libraries
import pandas as pd

%matplotlib inline

# Member demographics data set
members = pd.read_csv('meetup_data_set.csv', delimiter='\t')

```

Once we have the data in a DataFrame, we can now do a quick check to see the distribution of the **Location** information.

```python

members.Location.value_counts(ascending=True).plot(kind='barh', figsize=(12,14))

```

We can now create our geocoder using GeoPy. I used the GoogleV3 geocoder API for this project. Once again, you have to access a Google account and request the Developer API keys in order to use GoogleV3.

```python

# import geopy geocoder with Googlev3 API
# you need to get your own Google Maps Geocoder API key

from api_key import api_key # this is an api_key file I created to contain my key
from geopy import GoogleV3
geolocator = GoogleV3(api_key=api_key)

```

At this point, I wanted to create a separate file to store my geocoded location index. You do not have to do this to produce the geovisualization. However, since GeoPy sometimes has problems geocoding large numbers of locations (e.g., the aforementioned time-outs), doing this creates an index of unique locations with latitude and longitude I can use to create a offline geocoder.

```python

# create a list of all the unique locations
member_locations = list(members.Location.unique())

# create an empty list for the geocoder
mem_loc_list = []

# populate the empty list with the name, latitude, longitude
for i in member_locations:
    nom_geocoder = geolocator.geocode(i)
    lat = nom_geocoder.latitude
    lon = nom_geocoder.longitude
    mem_loc_list.append([i, lat, lon])

# import csv library
import csv

# write the geocoded locations to a csv file as backup index
with open('mem_loc_list.csv', 'w') as output:
    wr = csv.writer(output)
    wr.writerows(mem_loc_list)

```

Now that you have used GeoPy in a much longer form, here is a much shorter version that will create the geolocator, build a geocoded DataFrame with the location information, and then combine both DataFrames producing a geocoded members data set.

```python

from api_key import api_key
from geopy import GoogleV3
geolocator = GoogleV3(api_key=api_key)

# create a geolocator that will iterate through each row and geocode the location data for that row
mem_locs = [geolocator.geocode(i) for i in members.Location ]

# create a DataFrame with the location name, latitude, and longitude with matching headers to your original member DataFrame.
locs = pd.DataFrame([(i, i.latitude, i.longitude) for i in mem_locs], columns=['Location', 'Latitude', 'Longitude'])

# use combine_first to merge the two DataFrames based on matching headers and a one-to-one index.
members_geocoded = members.combine_first(locs)

```

Congratulations! You now have a geocoded data set that we can now put into a geovisualization. For this we will use Folium.

```python

# import folium
import folium
from folium.plugins import MarkerCluster

# create an object for Folium that contains the location informatio you want to map.
locations = list(zip(members_geocoded.Latitude, members_geocoded.Longitude))

# create a Folium Map object at a central location.
#Tell it which tileset you want as a basemap how zoomed in you want to be.
member_map = folium.Map(location=[40.403803, -49.219004], tiles='Cartodb Positron', zoom_start=3)

# add the location information from earlier to the Map object.
# Popups are the information from the members data set you want to render when
# a user clicks on a point, line, polygon, etc.
member_map.add_child(MarkerCluster(locations=locations, popups=members_geocoded["What do you think are the biggest issues in information analysis today?"]))

# save the file as an html file which will create the leaflet.js script for you
# you can also render this inline if you are using Jupyter Notebooks
member_map.save('member_map.html')

```

And now we have an interactive geovisualization that shows the thoughts of a spatially distributed community! You can play with the final product [here](https://thejoeblankenship.com/blogs/organizer_lessons.html "https://thejoeblankenship.com/blogs/organizer_lessons.html").

This functionality can be built into a dashboard or rendered as an entire page. In combination with BootStrap, it renders just as well on mobile as on a desktop. :)
