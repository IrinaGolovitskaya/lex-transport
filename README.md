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

At this point we have a chance to realize that the most apparant one is not always the best one. Further exporation and comparison with other datasets shows that it contains only bus stops for just a few bus routes ![values public transportation](graphics/public_transport_key.png). 
Although it contains important attributes (such as reference number and names of bus stops), this layer won't be used in the project. 
Another key to check is 'amenity', as it contains the value 'bus station". Access this data and get the polygon of the central bus station in Lexington and points just around it, which is also not useful for this project. 
So, the main source of data is the key 'route'. Using Quick quiery in QuickOSM plugin, I accessed the value 'bus' and acquire all the data. I have not chosen any other values, because there is no other public transportation system in Lexington. 

![QuickQuery](graphics/QuickOSM.png)
*Quick Query in OSM*

The route dataset contains bus route lines and definitive information, such as the start and end point, reference number of the route, its name and operator (well, there is only one operator in Lexington). Open Street map is a very rich source of data, but to obtain more accuracy it is good to check the consistency of information. The assumption is that the official website of local operator is the most reliable source of information. So I open [Lextran website](https://lextran.com) and check manually each route and its configuration (the OSM data appeared to be very precise). 

Hovewer, there are several considerations about the data filtering. 
* Route 8 changes due to the exact day of the week. 
Monday-Friday the route expands to BlueGrass airport. OSM data contains the shorter version, and I have chosen not to edit it, because the focus of my project is on general accessibility, and the stactic map gives very little opportunities of representing different schedules. 

![Route 8, Monday-Friday](graphics/Route8_MonFri.png)
*Route 8, Monday-Friday, official Lextran website*
![Route 8, weekend](graphics/Route8_Other.png)
*Route 8, weekend, official Lextran website*

* Route 27 won't operate till August, 2022. For purposes of interactive constantly updating map I would definitely filter out this route, but for the purpose of the static map there is no need to exclude anything that is out of operation temporarily. 

* I did not edit the OSM data to reflect any of temporary detouring or any kind of tempopary changes in schedules listed on Lextran website. 

* There are 4 night routes in Lexington. All of them are excluded from the analysis. As the final map is supposed to be static, not interactive, the night buses, which have different routes from the 'normal' ones, will significantly distort the analysis. 
