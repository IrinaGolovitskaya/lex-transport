# Public transport accessibility in Lexington, KY

## Table of Contents

- [Step 1. Accessibility](##step-1-accessibility)
     - [Data acquisition](###data-acquisition)
     - [Analysis](###analysis)
     - [Symbology](###symbology)
     - [Layout](###layout)
- [Step 2. Facilities](##step-2-facilities)
- [Conclusion](##conclusion-and-general-considerations)
- [References](##references)


## Overview

American cities and United States in general are well-known as a nation with a particular passion to automobiles. It does not necessarily mean complete abandonment of public transportation, but sometimes and in some places it may certainly be an issue. Especially it may affect quality of life of people with lower incomes and people who are coming to United States from abroad and may not even have driver's license at all. The rising prices may make a consideration of accessibility of public transportation even more relevant.
I decided to look at Lexington, KY public transportation system to see if the city in general is well-served and what is actually accessible and what can be done in the future. 
Last year I came to Lexington as an international student. As a person from abroad you have to visit many different local offices to get your documents done, so I decided to concentrate on accessibility of local offices that I newcomer in Lexington have to visit. 

## Step 1. Accessibility

But it makes no sense to start with a very specific locations on the map until we have an overall idea on how well Lexington is served with public transport, and how far we can go in specifying our analysis. 

### Data acquisition

Of course, at first we have to find (or create) reliable and detailed dataset. 

A quick examination of the [Open Street Map](https://wiki.openstreetmap.org/wiki/Main_Page) map feature documentation gives several possible options of acquiring data.
The most obvious one would be the key 'public transport' and all its values:

 * stop_position 
 * platform 
 * station 
 * stop_area. 

At this point we have a chance to realize that the most apparent one is not always the best one. Further exploration and comparison with other datasets shows that it contains only bus stops for just a few bus routes. ![values public transportation](graphics_readme/public_transport_key.png) 
Although it contains important attributes (such as reference number and names of bus stops), this layer won't be used in the project. 
Another key to check is 'amenity', as it contains the value 'bus station". Access this data and get the polygon of the central bus station in Lexington and points just around it, which is also not useful for this project. 
So, the main source of data is the key 'route'. Using Quick query in QuickOSM plugin, I accessed the value 'bus' and acquire all the data. I have not chosen any other values, because there is no other public transportation system in Lexington. 

![QuickQuery](graphics_readme/QuickOSM.png)
*Quick Query in OSM*

The route dataset contains bus route lines and definitive information, such as the start and end point, reference number of the route, its name and operator (well, there is only one operator in Lexington). Open Street map is a very rich source of data, but to obtain more accuracy it is good to check the consistency of information. The assumption is that the official website of local operator is the most reliable source of information. So I open [Lextran website](https://lextran.com) and check manually each route and its configuration (the OSM data appeared to be very precise). 

However, there are several considerations about the data filtering. 
* Route 8 changes due to the exact day of the week. 
Monday-Friday the route expands to BlueGrass airport. OSM data contains the shorter version, and I have chosen not to edit it, because the focus of my project is on general accessibility, and the static map gives very little opportunities of representing different schedules. 

![Route 8, Monday-Friday](graphics_readme/Route8_MonFri.png)
*Route 8, Monday-Friday, official Lextran website*
![Route 8, weekend](graphics_readme/Route8_Other.png)
*Route 8, weekend, official Lextran website*

* Route 27 won't operate till August, 2022. For purposes of interactive constantly updating map I would definitely filter out this route, but for the purpose of the static map there is no need to exclude anything that is out of operation temporarily. 

* I did not edit the OSM data to reflect any of temporary detouring or any kind of temporary changes in schedules listed on Lextran website. 

* There are 4 night routes in Lexington. All of them are excluded from the analysis. As the final map is supposed to be static, not interactive, the night buses, which have different routes from the 'normal' ones, will significantly distort the analysis. 

Thus, I save night buses as a separate geopackage. 
To filter all routes I use the following SQL expression:

```js
 "ref"='51' OR "ref"='52' OR "ref"='58'  OR  "ref"='59'  
``` 

Test shows 4 rows. I save it a separate geopackage in ESPG:3089 / Kentucky Single Zone projection.
The next is to filter all regular day buses: 

```js
  NOT ("ref"='51' OR "ref"='52' OR "ref"='58' OR "ref"='59' )
```
Test shows 37 rows (it is the right number - some routes consists of several lines). We save that as geopackage as well in ESPG:3089 / Kentucky Single Zone projection, because this is the prepared and filtered for the map data. 

Thus, the vector layer, that contains the bus routes is ready for analysis. But it is more appropriate to analyze the accessibility of public transport by buffering from the bus stops - people can only get to the bus at the bus stop, right?

So, according to Open Street Map documentation and quick examination of the attribute tables I have already acquired, there are three possible sources of bus stops points. It is highway, route_bus, amenity and public transport. Filter each point layer, get only bus stops in each one, and then use the Merge vector layer tool (**Vector > Data Management Tools**) to unite all the data(Route > bus_stop, Public transport > bus_station, Highway > bus_stop) in one layer for further analysis. Obtain something like this:

![Bus stops after merging layers](graphics_readme/Bus_stops.png)

It is well-known fact, that public transportation in many American cities leaves much to be desired, but to establish the route without stops is not something one could imagine. The only option available was to georeference lacking data, so I did that with the help of Google Maps XYZ Tiles. After long and boring night get this:
![Bus stops after georeferencing](graphics_readme/Bus_stops_after.png)

Using Quick Query I also access buildings (all values) and highways (all values). I save it in ESPG:3089 / Kentucky Single Zone projection. 
Now all the vector data is ready for the analysis. 

### Analysis

To define the accessibility of public transportation is to find out how many places one can reach using the public transportation network. 
[U.S. Department of Transportation](https://highways.dot.gov) in its pedestrian safety guide mention that people are generally ready to walk longer distance for railway stations or for leisure facilities, while they are ready to tolerate much shorter distance to a bus stop. Although access to diverse and qualitative leisure options may significantly affect quality of life, it is reasonable to start with basic. So for the purpose of this analysis I chose 5 min walking distance, or 1/4 mile (1320 feet). 
I go to **Vector > Geoprocessing Tools > Buffer**. 

![Buffer parameters](graphics_readme/Buffer.png)

And get the result:

![Buffered layer](graphics_readme/Buffer_result.png)

### Symbology 
Time to consider symbology. As I am considering accessibility, working with the output layer does not make a lot of sense. What really have to be shown is how many housing and facilities are located within those buffered zones. 
So, the main idea is to highlight those buildings (and here comes the 'building' layer obtained from OSM) which are within 1/4 mile distance and push back to the background those which are not. 
In order to do so, I duplicate building layer, and clip the copy with the buffered layer: 

![Clip](graphics_readme/Clip.png)

The clipped copy I symbolize with brighter colors, and the original one with darker colors and lower opacity. 
Generally, the choice of colors and symbols is inspired by the night sky, thunderstorms and lightning. It also help to visually divide the city spaces on the basis of public transportation accessibility. 

![Design](graphics_readme/Colors.png)

Colors used for the map:
* background: #1a162b
* bus lines: #e8ff91
* buildings within buffer zones: #4777e7 with the shapeburst fill to give it additional shine
* buildings outside buffer zones: #585b7d

The same colors I will be using for my web page to create a sense of consistency. 

### Layout

In the layout I used the sample colors for the title, map details, legend, the scale bar and the north arrow. Lexington's shape is hard to place neatly in most widely known page formats, so I chose to move all additional equipment to the left and zoom in the map itself. 
Also saved a template for additional maps. 

![The first map](Lex_transport_buffer_1200px.png)

## Step 2. Facilities

Now it is time for more detailed analysis. I use OSM to obtain locations of the local offices. I obtain the 'office' key using Quick Query in OSM plugin, then filter both polygon and points layers using the following expression: 

```js
"office" = 'government' OR 'association' 
```
I actually need both layers, because a certain offices are not duplicated. 
Then, there are many ways to check if these locations are accessible by public transport (the simplest one is just to look at the map), I used **Vector > Geoprocessing Tools > Intersection**. 
After symbolizing and labeling the new information, I get the picture like that:

![Facilities](graphics_readme/Facilities.png)

But visiting the coroner's office is not the best start of the new life in the city, is it? So, I remove all the information, that is unnecessary, and finally get a map of the state offices and its accessibility by public transport. 

![Final map](Lex_offices_1200px.png)

All presented offices are within the 1/4 mile distance, which was defined above as an appropriate distance from the bus stops. 

## Conclusion and general considerations

On the first map we can clearly see that there are entire neighborhoods that are underserved with public transport, and further consideration must be given to it terms if development the public transportation system. 
However, the local offices are accessible by public transport, so people who are new to the city and who have no car (in the college town like Lexington it is important) can easily and cheaply reach them.

There are several directions to improve, develop and elaborate this analysis. 
* Firstly, close examination of the bus routes in Lexington shows, that almost every route has its final destination downtown or close to downtown, it does not go through the entire city. That is why, even if the starting point of one's daily movements and the ending point are both within the walking distance from the bus stop, the whole journey may require one or two changing, which makes public transport intolerable option. The Network analysis on the scale of the entire city may help to bring important discoveries. 
* Bus schedules may be an issue as well, and in the advanced analysis of public transportation network it must be taken into consideration. 
* And finally, closer examination of the accessibility of local offices require consideration of the pedestrian infrastructure around them. There may be a bus stop nearby, but there may not be a pedestrian lane to the entrance, and that is the issue that people feel, but that is hidden on the small-scale map. 

### References
OpenStreetMap Wiki: https://wiki.openstreetmap.org/wiki/Map_features 
U.S. Department of Transportation oficial website: https://highways.dot.gov
Pedestrian Safety Guide for Transit Agencies issued by U.S. Department of Transportation: https://safety.fhwa.dot.gov/ped_bike/ped_transit/ped_transguide/ch4.cfm
Daniels, R., & Mulley, C. (2013). Explaining walking distance to public transport: The dominance of public transport supply. Journal of Transport and Land Use, 6(2), 5–20. http://www.jstor.org/stable/26202654
