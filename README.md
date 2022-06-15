# lex-transport

## Table of Contents

##

### Data acquisition

A quick examination of the [Open Street Map](https://wiki.openstreetmap.org/wiki/Main_Page) map feature documentation gives several possible options of acquiring data.
The most obvious one would be the key 'piblic transport' and all its values:

 * stop_position 
 * platform 
 * station 
 * stop_area. 

At this point we have a chance to realize that the most apparent one is not always the best one. Further exporation and comparison with other datasets shows that it contains only bus stops for just a few bus routes. ![values public transportation](graphics_readme/public_transport_key.png) 
Although it contains important attributes (such as reference number and names of bus stops), this layer won't be used in the project. 
Another key to check is 'amenity', as it contains the value 'bus station". Access this data and get the polygon of the central bus station in Lexington and points just around it, which is also not useful for this project. 
So, the main source of data is the key 'route'. Using Quick quiery in QuickOSM plugin, I accessed the value 'bus' and acquire all the data. I have not chosen any other values, because there is no other public transportation system in Lexington. 

![QuickQuery](graphics_readme/QuickOSM.png)
*Quick Query in OSM*

The route dataset contains bus route lines and definitive information, such as the start and end point, reference number of the route, its name and operator (well, there is only one operator in Lexington). Open Street map is a very rich source of data, but to obtain more accuracy it is good to check the consistency of information. The assumption is that the official website of local operator is the most reliable source of information. So I open [Lextran website](https://lextran.com) and check manually each route and its configuration (the OSM data appeared to be very precise). 

Hovewer, there are several considerations about the data filtering. 
* Route 8 changes due to the exact day of the week. 
Monday-Friday the route expands to BlueGrass airport. OSM data contains the shorter version, and I have chosen not to edit it, because the focus of my project is on general accessibility, and the stactic map gives very little opportunities of representing different schedules. 

![Route 8, Monday-Friday](graphics_readme/Route8_MonFri.png)
*Route 8, Monday-Friday, official Lextran website*
![Route 8, weekend](graphics_readme/Route8_Other.png)
*Route 8, weekend, official Lextran website*

* Route 27 won't operate till August, 2022. For purposes of interactive constantly updating map I would definitely filter out this route, but for the purpose of the static map there is no need to exclude anything that is out of operation temporarily. 

* I did not edit the OSM data to reflect any of temporary detouring or any kind of tempopary changes in schedules listed on Lextran website. 

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

So, according to Open Street Map documentation and quick examination of the attribute tables I have already acquired, there are three possible sources of bus stops points. It is highway, route_bus, amenity and public transport. Filter each point layer, get only bus stops in each one, and then use the Merge vector layer tool (**Vector > Data Management Tools**) to unite all the data in one layer for further analysis. Obtain something like this:

![Bus stops after merging layers](graphics_readme/Bus_stops.png)

It is well-known fact, that public transportation in many American cities leaves much to be desired, but to establish the route without stops is not something one could imagine. The only option available was to georeference lacking data, so I did that with the help of Google Maps XYZ Tiles. After long and boring night get this:
![Bus stops after georeferencing](graphics_readme/Bus_stops_after.png)

Now all the vector data is ready for the analysis. 

Using Quick Query I also access buildings (all values) and highways (all values). I save it in ESPG:3089 / Kentucky Single Zone projection. 