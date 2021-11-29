
# FerrarGIS

*A modern-day Ferraris atlas*

Using QGIS to apply a 1777-inspired style to today's OpenStreetMap data. 

![FerrarGIS map of Ghent](examples/gent_a5_landscape_ferrargis.png)

![1777 Ferraris map of Ghent](examples/gent_a5_landscape_1777.png)

In this project I explore if it's possible to recreate the iconic style of the Ferraris map with modern GIS tools and open data: *What would it look like if count de Ferraris would have made his map today?*

This Readme contains detailed information about the design process and results, and how to set this up yourself using QGIS and the `.qgz` project document included in this project. I've tried to explain everything in a pretty verbose way here, hoping to help fellow FOSS4G users, share some lessons learned and inspire others in their creative endeavours.

## Approach

In my earliest experiments I explored tools like CartoCSS and the Mapbox Style Specification, hoping to efficiently create a webmap of the entire Belgian territory in a Ferraris style. But I quickly realised that both were lacking some crucial features and effects to come close enough to the specifics characterising the Ferraris maps. 

My eventual setup was much more desktop oriented: a Postgres database filled with the latest OpenStreetMap download, loaded in QGIS and styled while taking advantage of QGIS's powerful symbology and labeling features.

## Data

The data for this project comes from OpenSteetMap: and Open Source map (and database) of the entire world. The data was downloaded in September 2021 from the *Geofabrik* OpenStreetMap Data Extract for Belgium, and loaded into a Postgres database. This creates three database tables containing points, lines and polygons respectively describing features like trees, roads and houses. QGIS allows one to load these tables as layers, and style them as desired. The various properties of the features in these tables are described using key-value pairs, and these can be used to create layers for specific features (e.g. one can make a 'house' layer by applying the properties filter `"building" = 'yes'` on the polygon layer). The [OpenStreetMap Wiki](https://wiki.openstreetmap.org/wiki/Main_Page) was a great help to navigate the most commonly used keys and tags, as selecting all relevant features for that needed to be styled was quite a tedious undertaking: excluding underground roads, rails and buildings, including all types of historical buildings, excluding marine parks, etc.

## Styling

The Ferraris maps have a very recognisable style, but not every Ferraris map sheet looks the same. First of all, there are three copies of this map (in the national archives of Vienna, Amsterdam and Brussels respectively), each with a very different style. I went with the Brussels one as it is the most well known: it was recently published as a hardcover atlas by the Belgian Royal Library KBR and it has even been georeferenced (and made publicly available as a WMS service) by the Belgian regional cartographic institutions. But this choice alone is not specific enough: there are small but important stylistic differences between the 275 different map sheets, noticeable in the color hue, brightness and saturation. As I was looking for reference imagery, I would for example notice the tree or grass color would differing remarkably between the map sheets of Ghent and Brussels. I ended up choosing the Ghent map sheet as my main point of reference and selected my sample pictures and colors from this sheet are much as possible.

To make the QGIS style-sheet resemble the original Ferraris map as much as possible, I used carefully sampled colors and fill-images in the style definitions of the drawn objects. To obtain the right fill colors for things like buildings, grass and water polygons, I used a color picker tool to find the right colors on the old map. For more complex features like woodlands, marshes and heath that are drawn with beautiful textures and miniatures on the old map, I took a screenshot of such these drawings, used a graphics editor to clean up the images and make them seamlessly tileable, and then used them as the background images for the corresponding features.

### Advanced styling notes

I used used quite a few advanced QGIS features to arrive to the final style, so I'll share some of them here for inspirational purposes:

- All sizes (e.g. line width) are specified relative to the map units, not to the screen pixels (as is most common). For example, roads are drawn 7 meters wide, independent of the zoom level. The Ferraris maps are made at a very specific scale (`1:11.520`), and I wanted my map features to be drawn at the same size at any zoom level.
- Buildings, rivers and streets are drawn as 'merged feature': all features are merged into one before they are styled. This was necessary to obtain the characteristic merged-styling of houses on the 1777 map, but also to prevent seams between water surfaces and obtain connected street layouts.
- Buildings in particular use another great QGIS feature: in the 'draw effects' an inclined inner shadow is specified to give them the same height-effect as on the Ferraris maps.
- To deal with the fact that the OSM dataset does not contain outlines for *all* houses (specifically in more remote villages), I used a screenshot of a village area on the old map as the background fill for residential areas. The image contains small fields, trees and some houses. In residential areas where OSM does contain outlines for the houses, I draw the houses on top of this image, surrounding them with a soft neutral halo to make them blend in with the background picture.
- On Ferraris' map, important buildings such as castles and churches are drawn in a darker red. To obtain this, I put an extra layer of all houses whose `historic` field was specified, or who's building type was church, cathedral or abbey. Just like on the 1777 map, I drew a cross over the churches, aligning the cross with the building by defining it's angle using the `main_angle()` function, which specifies the main angle of the smallest rectangle completely covering the feature. This approaches also came in handy to orient the line patterns on the agriculture fields.
- The OpenStreetMap dataset contains some trees, either mapped as tree, as tree row or as hedge. I added all these to the map as a small circle. I also added some extra trees by drawing a buffer around the farmland's edges and filling it with some random trees. This makes the map look a lot more 'real', and may compensate for the many actual trees that are not in the dataset, but I'm also aware this alters the data in a way that should be evaded.
- Next to buildings, water is a distinct feature on the Ferraris' map. It was quite a challenge to replicate the various ways water is displayed in rivers, ponds and streams, since the colors, gradients and borders used differ between similar water features. I ended up applying a centroid fill with a carefully picked gradient. This yielded a close enough resemblance for most features, and is also quite a pleasing sight. 
- A final trick I used to make the map look *old* was to add a layer applying the 'speckle' found on many old maps. To recreate this, I used a screenshot of an empty map page of the Ferraris map and used GIMP to make it into a pure speckle layer: making it black-and-white and focusing it's thresholds on the speckle. I then added the layer using the `multiply` setting in the draw effects, which resulted is a subtle, realistic speckle.

### Typography

The typography represented another opportunity to create something close to the old map. I identified the *Youngbae* font family as close to the typography used by Ferraris, and specified the labels for the towns, villages and  and rivers. The labels were given a slight incline just like on the old map, and a 1 pixel semi-transparent black buffer to make them a little more fuzzy. They were also set to evade built areas and forests for optimal legibility.

## Exports

I used the QGIS composer and it's Atlas settings tool to export big A1 images of some major towns and make many smaller A5 export of towns, villages and more remote areas. To be able to compare my map with the original one, I also made exports of the same areas of the 1777 maps, i.e. the stitched together and georeferenced maps that are made available as WMS by the Belgian regional cartographic institutions. Note that my maps don't replicate the original 1:11.520 scale of the Ferraris maps, but since I made my maps scale-independent, that just means they are photographically zoomed in or out - which is also the case for the exports of the original maps!

## Result

I'm proud of the final result. It's not perfect, but I think it succeeds as a thorough exploration of recreating the old style with modern techniques and data.

### Difference with the 1777 map style

There are some differences between my results and the 1777 map I would like to enumerate here. Some could be solved in future iterations, others are more inherent to the project itself. A first one, for example, is the language: the old map used the (18th century) french names for towns and villages, and named features like farms, abbeys and quarries with their french terms. It also put red numbers next to such features to indicate the parish they belonged to, something I didn't replicate. While I did add specific drawings for windmills, watermills and trees, the original map included many more drawings, for things such as gallows, battlefields, pillories and archery targets.

There are some polygon features I could have styled but didn't, either because I didn't find a corresponding image on the old map or because I didn't like the result. Such areas include rocky areas, shallows/fords and, most notably, pine forests. Something I consciously chose to not reproduce was the styling of footpaths: Ferraris used dotted lines for these, but I found this to create many undesirable effects in city parks and cemeteries and thus displayed all roads with beige lines. Furthermore, I was hoping to be able to recreate the beautiful arrows used to indicate river currents, but all my attempts failed. Finally, the 1777 map didn't have some modern features like railroads and tramlines, for which I chose to invent a suitable style.

While looking at the exports more recently, I also noticed some areas with imperfect styling, including brownfields, hospitals and the royal park. Feel free to contact me if you notice any imperfections or have ideas on how to improve this map.

### Relief shading

The relief shading on Ferraris' map is also a recognisable stylistic feature. I tried a couple of approaches to reproduce it, but haven't been able to do so so far. Ferraris styled slopes and hillsides in a brown gradient that becomes darker near the top. These slope lines are different from the classic contour lines you can find on more modern topographic maps, as they are not continuous nor occur at specif heights. To reproduce such slope lines, I needed some way to locate all slopes in Belgium. Sadly, OpenStreetMap dataset does not contain general height data - only the special case of steep cliffs are mapped. There are some digital terrain models of Belgium available to the public, and one could write a script that computes the average/smoothed slope everywhere from these serrated terrain models, and extracts the main slope lines from it, but such an approach seemed out of the scope of this projects, so for now only the cliffs can be found on the map (styled as brown slopes) and general relief shading is missing.

## Getting started

To set this project up yourself, download QGIS v3.20 or higher, and download the OSM data of Belgium to a Postgres database names `osm_be` (for example, using the `osm2pgsql` tool, as explained in [this tutorial](https://learnosm.org/en/osm-data/osm2pgsql/)). You should then be able to open the `.qgz` project file in QGIS. Note that it may help to first open a new QGIS project, zoom in to a small, village sized area and disable rendering before opening the project.

## Useful links

- The OpenStreetMap [Standard tile layer](https://wiki.openstreetmap.org/wiki/Standard_tile_layer)
- Champs Libres's [blog post](https://www.champs-libres.coop/blog/post/2019-12-17-style-qgis-pour-osm/) on developing a OSM stylesheet for QGIS

## Credits

The original maps are courtesy of [GeoPunt - Agentschap Informatie Vlaanderen](https://www.geopunt.be/catalogus/datasetfolder/2d7382ea-d25c-4fe5-9196-b7ebf2dbe352) and [GeoPortail - Service Public Wallonie](https://geoportail.wallonie.be/catalogue/b8b9e555-a4d1-49bf-940d-31bbbf7613fc.html)

## License

This map style is licensed under the Creative Commons CC BY-NC-SA 4.0 license.

Please mention the following source:

> Map style by Manuel Claeys Bouuaert, distributed under CC BY-NC-SA.

## Contact information

Have ideas on how to improve this map, or want to use it in a project or exhibition? Feel free to contact me at [manuel.claeys.b@gmail.com](mailto:manuel.claeys.b@gmail.com)
