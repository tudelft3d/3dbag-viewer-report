# Requirements for nation-wide 3D citymodel webviewer
> Written by Ravi Peters & Stelios Vitalis ([3D geoinformation group](https://3d.bk.tudelft.nl), TU Delft)

This documents describes the requirements for a 3D webviewer for sharing a nation-wide 3D citymodel with the general public. 

A 3D citymodel is a dataset that contains objects that are commonly found in cities such as buildings, roads, vegetation, terrain etc. Building objects are generally seen as the most prominent feature type of a 3D city model. Contrary to its name a city model does not always refer to the extent of a city and can also contain parts that are not strictly part of a cit (e.g. covering rural areas). In fact, this document focuses on nation-wide 3D city models. Such a dataset contains objects for the whole country. The next iteration of the 3DBAG dataset and the new 3D Basisvoorziening by Kadaster can be considered such nation-wide 3D city models.

*Why make an online 3D viewer for a nation-wide city model?*

The 3D Basisvoorziening is an open dataset that is generated and maintained for the public good from public money. Therefore it should also be publicly accessible. First, this means that the data needs to be disseminated in a usable data format so that everyone can easily download and use the data. Furthermore, considering the novel 3D aspect of a 3D city model (compared to other public datasets), it is also important to lower the barrier of discovering and exploring the data as much as possible. A user should be able to quickly see what such a dataset contains and what new possibilities it offers, without needing special technical skills or know-how.

A 3D webviewer, when implemented well, is a very effective tool to encourage such exploration and discovery of the nation-wide 3D city model. One only needs to click on a link to open and interactively explore the dataset directly on their device. The main benefits of having a 3D webviewer are:

* A 3D citymodel is best understood in a 3D environment.
* Lowest possible barrier to access and see the nation-wide 3D city model. No specific technical knowledge or skills required.
* It is good for publicity if done well.
* It can serve as a platform to demonstrate what is possible with 3D (use in decision making/planning, show case results of environmental simlations like flooding, noise, etc) and to support development of further 3D tools.

In the remainder of this text we will look at what is needed to make such a viewer succesful, discuss the relation of the  viewer to other parts of the dissemination infrastructure, and discuss the available technical components that are presently available and used. A summary with the main conclusions and recomendations is given at the end.

## 1 Viewer requirements
In order to understand what defines a succesful and good 3D web viewer, we need to understand its requirements. We categorise requirements according to how important they are.
<!-- *General requirements on the viewer, desired functionalities, viewer components* -->

#### High priority requirements [H]
These mostly non-functional requirements need to be met in any case. A viewer that does not implement these requirements in a good manner will not be succesful.

1. *Ability to view the buildings of the complete nation-wide 3D dataset in LoD1.2 or higher in a highly performant manner*

	Minimally with a 2D map as base layer. Must be performant and usable on mobile platforms. Important to minimise required network bandwidth and minimise computational load on the client, so that the user has a fluent experience without unnessecary delays or hiccups.

2. *Intuitive and easy to use*

	Keep the user interface and controls as simple as possible. People lose interest if things are unnecesarily complicated or unclear. The main audience of the viewer should be a general user and not a domain expert of 3D city models.

3. *Open source* 

	Both the used data/exchange formats and the viewer code itself must be open source. This encourages others to use the data and build their own functionalities with the viewer and contribute back their improvements.
    
4. *Address lookup (geocoding)*

    The user types an address and the viewer jumps to the corresponding geographic coordinates.
    
5. *Ability to easily download data in a usable format*

	Download city objects by simply drawing a 2D (x/y axis-aligned) rectangle in the viewer for the area of interest. The viewer applications forwards this 2D rectangle to a download service, which collects the intersecting objects which can then be downloaded by the user. File format for the download should be CityJSON, but other options that are widely supported in 3D software (eg .OBJ) could also be considered (a possibly concern is that such formats do not support semantics and geometric coordinates).

#### Medium priority requirements [M]
These functional requirements make the viewer more appealing by introducing more features without increasing the complexity of the 3D scene. Therefore they will have minimal impact on the viewer performance, while stil greatly improving the appeal and usefulness of the viewer.

6. *Ability to query semantic information for city objects eg. by clicking on them*
	
	The user must be able to request and see all the attributes (semantics) of a city object that are contained in the 3D city model dataset itself. In this way the user can inspect all aspects of a city object (both 3D geometry and attributes) directly from the viewer.
    
7. *Ability to query semantic information for city objects from other related datasets*

    This is a natural extension of the previous requirement. It allows the user to retrieve attributes/information about a city object from other related datasets that contain relevant information. This should ideally happen through a separate API that disseminates the related dataset. Notice that this separate API is not part of the viewer infrastructure itself, but the viewer can connect to it and use it (see Section 2). This is in accordance with the [Dis-Geo](https://www.digitaleoverheid.nl/nieuws/doorontwikkeling-van-basisregistraties-in-samenhang-dis-geo/) idea.

8. *Ability to switch between different LoDs for buildings*

	For LoD1.2, 1.3, and higher if available. At any time only one LoD is shown.


#### Low priority requirements [L]
These are functional requirements that are nice to have in the long term. They have the potential to make the viewer significantly more interesting and useful, but when implemented poorly the performance of the viewer may deteriorate. Therefore these requirements should be carefully considered and implemented.

9. *Ability to visualise properties of objects*

	A simple example is to color the buildings according to some attribute/semantics (see for example this [SPOTinfo video](https://www.youtube.com/watch?v=kc65iBk7YBU)). This could be semantic information directly contained in the 3D city model itself, or from a linked dataset (see requirement M8). Preferred are use cases where 3D has real added value like facade level noise simulation results (very difficult to visualise on a 2D map).

10. *Ability to see nation-wide 3D Terrain*
	
	After 3D buildings are visualised, the visualisation of 3D terrain is the next logical step to be achieved. The simplest and most performant way to achieve this is to take a 3D mesh with a 2D map containing the objects on the ground draped on it as a texture. This will work well with different terrain LoDs. Separate 3D terrain objects are technically also possible but are more complicated to get right with different LoDs and may most likely have a bigger impact on performance.

11. *Ability to see other 3D objects than buildings and terrain*

	For example vegetation/trees, viaducts, and bridges, etc. Depends on the availability of such data, but also the complexity of the objects. Adding too many complex objects will compromise the performance of the viewer.

## 2 Technical infrastructure
This section gives a brief overview of the complete infrastructure in terms of databases, API services and webapplications that are needed to realise the 3D webviewer. 

The image below illustrates the main components and how they are linked to each other.

![Infrastructure overview](https://i.imgur.com/muyzHTV.png)

The viewer infrastructure itself is highlighted in blue and consists of the following parts:

1. **(PostGIS) Database** that contains the current version of the 3D city model
2. **Static geometry files that are optimised for consumption by the viewer**. This is essentially a static copy of the geometries in the database in a format that can be quickly transferred and loaded into the viewer. Likely to use some kind of tiling scheme so that the viewer can easily load only the parts of the city model that are in the current view (to maximise performance and minimise how much data needs to be downloaded).
3. **Feature API for the 3D city model**. This is a dynamic service that can be queried by client applications (our viewer application, but possibly also other applications). Through this service any part of the city model can be requested/downloaded in a format that is good for dissemination. A download feature in the viewer would utilise this API.
4. **Viewer application**. This is the webapplication that runs in the browser of the user. It is primarily responsible for rendering the 3D city model and the user interface in the browser window.

In addition we should consider the integration of the viewer infrastructure with external APIs and applications shown in grey in the diagram (development of these is outside the scope of the viewer):

1. **External Feature API's** that deliver additional information about the city objects that is contained by external related datasets (see requirement M7). These external APIs are shown in grey in the figure. Using such a mechanism we could display attributes from external datasets in the viewer when the user clicks on an object. This would require that such Feature API services exist and that the respective data models are designed such that this linking is indeed easy to do (for instance using a shared object identifier).
2. **Other applications** can connect to the Feature API for the 3D city model to download city objects.


## 3 Current state of the art
This section gives a non-exhaustive technical overview of the most popular currently available standards and libraries that can be used to realise a 3D web viewer.

### Data formats/standards

Several data formats are available to be used for such a solution. Originally, 3D data were mostly the focus of the 3D graphics domain, but lately more integration with geo-information data has been achieved.

Below, is an overview of formats and standards that can be used by a web viewer for 3D city models. Not all listed technologies are appropriate for immediate consumption by a 3D viewer, but some (e.g. CityJSON and/or 3DcityDB) are important for the understanding of 3D city models and can be used as intermediate way to store and transform data.


#### Optimised for visualisation and network exchange
#### [glTF](https://github.com/KhronosGroup/glTF/blob/master/README.md) 

A specification built by the Khronos group (the one that specifies open graphics specs such as OpenGL, WebGL, OpenCL, Vulkan etc.) for dissemination of 3D graphics data for visualisation. It is mainly developed to accomodate WebGL on the efficient distribution and storage of 3D graphics data.

Its main focus is highly efficient 3D data exchange for visualisation (e.g. games) and there is minimal support for any semantics. No CRS or other geographic information are supported by the standard.

Basically, a `gltf` file is a JSON-file that defines a scene graph. The assets of the graphs (3D models) are stored in binary formats also defined by the glTF specification.

#### [3D Tiles](https://github.com/CesiumGS/3d-tiles)

A specification originally developed by Cesium (previously AGI) for dissemination of massive 3D geoinformation datasets, with focus on visualisation. Today it is an OGC [standard](https://www.ogc.org/standards/3DTiles). It defines two things:

1. a tiling mechanism for 3D data, with functionality such as LoD (mostly refered to as "geometric error"), and
2. data formats for storing and exchanging 3D geodata.

The tiling specification is extensive and provides a wide array of features. For a typical distribution of 3D city objects it is mostly overwhelming: the tiling aspect of a nation-wide 3D city model can be mostly served by a 2D scheme (especially in the Netherlands). 3D tiles' complex specification is more appropriate when distribution across all three dimensions is important (e.g. point-clouds) or when used for heterogenous data (e.g. combination of 3D models and point-clouds).

Tiling is mainly done by a JSON file (tileset.json) which specifies the bounding box, data and other information for every tile.

Four (4) formats for the exchange of the 3D data are defined:
- `b3dm`: which can describe 3D models with attributes (basically a wrapper around the `gltf` binary format, mentioned before).
- `i3dm`: similar to `b3dm`, but for instanced models (e.g. trees) where the geometry is defined once and then rendered multiple times.
- `pnts`: for point-cloud data.
- `cmpt`: for composite objects (combinations of the above).

3D Tiles' main benefit is that it can support the basic requirements for BAG 3D data. But it requires a complicated toolchain to be built in order to be able to produce such a dataset. The specification is too complex for a simple distribution of city objects across the country. Nevertheless, data is stored and distributed in a binary format which makes them more efficient (less network bandwidth is used).

There is software to create 3D tiles. But the most robust solutions are provided from Cesium. They implicitly push people to use their own services, e.g. most examples refers to their proprietary services (Cesium Ion). There are other open-source implementations for creating 3D tiles (e.g. [pg2b3dm](https://github.com/Geodan/pg2b3dm/)), but mostly sparse functionality is provided and reliability is questionable.

#### [i3s](https://github.com/Esri/i3s-spec)

A specification by ESRI for the delivery of complex 3D scenes, which is an OGC community standard for dissemination of 3D geoinformation data, with focus on visualisation. Intends to basically serve the same purpose as 3D Tiles, but offers a more complex and feature-rich solution to it.

The main difference between i3s and 3D Tiles is that the first mainly focuses on the delivery of geometric information, while the semantic aspect of the model is served on-demand (only when necessary). That is contrary to the fact that 3D Tiles serves both semantics and geometries in one chunk per tile. This maximises the number of network requests, but minimises the amount of data originally required to view the geometries.

While i3s have been around for as long as 3D Tiles, there is far less use to it in 3D city model delivery. An implementation of the i3s would also require more work than 3D Tiles, given that the latter reuses the glTF technology which is "native" for 3D web viewers of all kinds.

#### [CityJSON](https://www.cityjson.org/)

An open specification mainly developed by the 3D geoinformation group of TU Delft, which is in the process of being established as an OGC community standard. Its main purpose is the storage and exchange of 3D city models. Basically, a JSON encoding of the CityGML data model, which describes city objects with 3D geometries, attributes and semantic information.

It provides complex features, such as the ability to have attributes per surface of a 3D object (e.g. a roof of a building), multiple textures/materials etc. Certain optimisation techniques are utilised, such as compression of vertices (minimises storage and standardises precision of coordinates). Metadata support is extensive (CRS included).

The specification is built to be used mostly for the storage of data per city region. Due to its use of indexed vertices, it cannot be used to distirbute parts of the region. Should be the most storage-expensive alternative of all, as well. It is the least efficient for the use with a nation-wide web viewer, but is the best data format to store the 3D city models on the server.

#### OGC API Features ('WFS3')

The successor of the well known WFS standard. An OGC standard for an on-demand service that returns data according to the user's request parameters (REST API) for dissemination of geoinformation data, with focus on dynamic queries and consumption from GIS clients. It is format agnostic, meaning that data can be returned in any format (CityJSON, CityGML etc.).

Notice that this is not a file format as the ones listed above, but an API to deliver files. As such it would be a good fit for implementing a download servive (requirement H5).

Currently, there is experimental work on an implementation for CityJSON. Regarding bandwidth, it potentially be decent enough, but it requires constant calculation from the server in order to compute and produce the result. Suitable for creating a download service, and likely to be widely supported by many software in the future.

#### [3DcityDB](https://www.3dcitydb.org/3dcitydb/)

A schema to store CityGML information, developed by virtualcitySYSTEMS, to store 3D city models in a database. Can be used with PostgreSQL or Oracle databases. There is a tool to load data from CityGML/CityJSON with a GUI (all Java technology). It is also possible to export data in 3D Tiles.

### Viewers

#### [three.js](https://threejs.org/)

A low-level framework to ease the development of 3D web application. It utilises WebGL and is the most flexible option, given that manipulation of data is done directly in the lowest level (meshes, textures, materials etc). It has been around for years and is the most popular choice for a web 3D graphics engine.

There is no GIS/tiling functionality built-in. Any data format of the above needs to be implemented for the specific use case. For the moment, there is [ninja](ninja.cityjson.org) that uses it with CityJSON and certain aspects of the code could be reused (see further). No robust and reliable solution exists for 3D tiles or any other tiling mechanism whatsoever. There is a promising open-source project, though, from [NASA AMMOS](https://github.com/NASA-AMMOS/3DTilesRendererJS).

It can be used to produce very pleasing viewers, such as [f4map](https://demo.f4map.com/) (not open source), or [vizicities](https://raw.githack.com/robhawkes/vizicities/dev/examples/colour-by-height/index.html).

<!-- Might not be relevant:  http://vcg.isti.cnr.it/nexus/ -->

#### [CesiumJS](https://cesium.com/index.html)

Developed by Cesium, which also created the 3D tiles specification. It is basically a 3D globe with the ability to show 3D geoinformation from multiple formats.

Functionality is extensive and many GIS features are available (e.g. support for CRS). Yet, the software is bloated and needs extreme fine-tuning to achieve decent performance. Also, all implementations look alike and the result is not necessarily the most eye-pleasing for the general audience.

Most famous 3D city model's viewer implemented with CesiumJS is **virtualcityMAP**, from virtualcitySYSTEMS.

Multiple agencies and organisations use it, mostly because of the out-of-the-box support for 3D tiles and because virtualcitySYSTEMS promotes it. Examples:

- [SwissTopo](https://map.geo.admin.ch/?layers=ch.swisstopo.swissnames3d&lon=8.24528&lat=46.04722&elevation=87928&heading=360.000&pitch=-44.188&lang=en&topic=ech&bgLayer=ch.swisstopo.pixelkarte-farbe)
- [3D Rotterdam](https://www.3drotterdam.nl/#/)

#### [Mapbox GL](https://docs.mapbox.com/mapbox-gl-js/api/)

A mapping library to build viewers that primarily consume Mapbox data. Focuses on 2D maps and styling capabilities. There is limited out-of-the-box 3D viewing functionality: polygon extrusion by a height-attribute, which is good for LoD1.3 but not useful for LoD2. One can also create layers using three.js directly to import/visualise complex 3D models, but this requires a lot more tinkering.

Default functionality is very efficiently implemented, also on mobile platforms, and easy for the eye for regular users. There is [experimental](https://github.com/Geodan/mapbox-3dtiles) support for 3D Tiles (useful for LoD2). 3D terrain currently is not supported, but it seems that Mapbox is looking into it:
- [3D terrain rendering](https://blog.mapbox.com/bringing-3d-terrain-to-the-browser-with-three-js-410068138357)
- [3D terrain simplification](https://observablehq.com/@mourner/martin-real-time-rtin-terrain-mesh)

Examples:
- [LoD1 buildings colored by building age](https://parallel.co.uk/netherlands/#13.8/52.365/4.9/0/40)
- [LoD1.3 buildings](https://3d.bk.tudelft.nl/opendata/noise3d/lod13map.html)
<!-- Multiple higher-level frameworks to work directly with data, such as: [deck.gl](https://deck.gl/#/). -->

#### [harp.gl](https://www.harp.gl)

It is similar to MapBox, but for Here maps (Microsoft). This is slightly more low-level than MapBox, though. It has pretty much similar benefits and drawbacks with Mapbox, but is far less popular. 

#### [ninja](https://github.com/cityjson/ninja)

A CityJSON web viewer. It uses Vuejs for the GUI and three.js for 3D visualisation and focuses a lot on the semantic aspect of the model (objects' hierarchy, details of the content of an object).

There is currently no support for terrain, except for TINRelief in CityJSON. It is also intended to work as stand-alone, without a backend. The user loads their own data in the viewer. There are plans to change this, though, and it should be easy to adopt it to a visualiser of OGC API Features.

The vue components are reusable through [cityjson-vue-components](https://www.npmjs.com/package/cityjson-vue-components). So it should be easy to build a new viewer with some of the parts of ninja for the purposes of BAG 3D.

## 4 Three scenarios for implementing the viewer

Based on the available state-of-the-art tools we listed before, we propose three general scenarios to implement a viewer.

### Scenario 1: CesiumJS + 3D tiles

Delivering data as 3D Tiles and consuming through CesiumJS seems like the most straightforward approach for such a viewer. It is the easiest way to start (no wonder why most agencies use it already), but also have the biggest drawbacks in performance and usability (which is the reason why people don't really use them that much).

#### Pros
- Those technologies play very well with each other (both Cesium products).
- It is easy to start with this combination. We managed to build a [prototype](https://github.com/liberostelios/cesium-prototype) in less than an hour, using FME to create 3D tiles (not that reliable) and following their [guide](https://www.cesium.com/docs/tutorials/cesium-and-webpack/).
- 3D Tiles ensure good enough performance, regarding network transactions. Data exchange speed will never be too bad.
- CesiumJS ensures certain extent of geo-features out-of-the-box:
    - easy support for terrain with drapped images from multiple sources,
    - styling per attribute,
    - support for many GIS and 3D formats.

#### Cons
- Requires that 3D tiles are somehow created as a cache of the data (that is both good and bad). There are many ways on how to do that, but the best one is to use Cesium's proprietary products and services.
- Cesium seems to promote their services a lot. Even in their documentation it becomes difficult to designate which parts are the open source/standard technologies and what are their proprietary products (e.g. they push a lot Cesium Ion, their own data delivery service).
- 3D tiles are open standard and famous, but still the most reliable solution is using Cesium's propriatery software/services.
- Vendor lock-in is still a minor possibility, based on the previous. While all is open source/standards, the community has not gone too far to overcome the software monopoly of Cesium in the ecosystem.
- Performance can never be great. While web transactions are optimised, for most computers using a single CesiumJS viewer might be challenging. So building any sophisticated UI using a JavaScript front-end would definitely make performance worse. That limits the choices of GUI and requires higher amount of time for optimisation to deliver a useful final app.
- Final product looks always the same. You can tell that a tool is based on Cesium from miles and that is not a good thing, because it looks boring and does not attract the user (that is, also, due to the dissapointing performance).

### Scenario 2: Mapbox GL + 3D Tiles

MapBox is better performing than CesiumJS. Nevertheless, there is no reliable and complete solution for using 3D (Tiles) with it, except for a [prototype](https://github.com/Geodan/mapbox-3dtiles) developed by Geodan. Further development towards this direction would be needed. Mapbox's power would show when using 2.5D data like LoD 1.x and no 3D terrain (so, practically when not using any 3D geometries at all). Implementing a full 3D solution is challenging since the viewer is not intended for consumption of truly 3D data. For example, in Geodan's prototype a terrain is visualised, but collides with the existing 2D mapping plane of the viewer.

#### Pros
- Lightweight and high performance, also on mobile (mostly).
- Optimised for data viewing in simple use cases, where footprint height can be used. This can be used to have a robust viewer for LoD1.x buildings.
- Possible to extend with more 3D features by implementing a three.js layer
- Easy to use and adapt. Good styling mechanism, user friendly.

#### Cons
- Only supports 2D geometry sources out of the box.
- Support and features for complex 3D geometries still lagging (e.g. for LoD2). No reliable solution for 3D tiles, so custom implementation/maintenance of a 3D tiles parser will be necessary.
- Not really possible to elegantly implement 3D terrain since there is always a 2D baselayer.
- Vendor lock-in is a possibility, maybe slightly more prominent than Cesium. They promote their own proprietary services. Using their own open software/standards (e.g. vector tiles) is very hard without their support.
- Performance is not guaranteed if one wants to use it for more than LoD 1.x, since custom code has to be written for 3D tiles (and complex 3D geometries in general).

### Scenario 3: three.js + 3D tiles/OGC API

Three.js is by far the most low-level framework of all. A custom-made solution using three.js would require manipulation of all graphics aspects (scene management, lighting, camera etc.) which is more complex. Yet, performance can be as good as the time invested in the development. Also, efficient use of shaders/post-processing effects could make a far more good-looking 3D output than anything else, without sacrificing performance.

#### Pros
- Rendering quality and performance possibilities are endless (could look as good as a game). A good implementation could look and perform as good as [f4map](https://demo.f4map.com/) does.
- No bloated functionality or redundant features. Everything will be tailored to the requested needs (which allows better performance).
- Plethora of [examples](https://threejs.org/examples) and prototyped code to combine in order to achieve certain results, both visually and for geo-processing (e.g. [CityJSON Vue components](https://github.com/tudelft3d/cityjson-vue-components), [3DTilesRendererJS](https://github.com/NASA-AMMOS/3DTilesRendererJS)).
- Can be more dynamic. For instance, 3D tiles and OGC API can be used interchangeably to re-use parts of the delivery service.

#### Cons
- Most things have to be implemented from scratch, for example:
    - terrain support
    - management and parsing of tiles
    - camera movement
- Performance could be abysmal, if things are not implemented right.
- Higher maintenance cost. Bugs will appear in functionality that otherwise comes out-of-the-box with CesiumJS or MapBox GL.

### Comparison of the scenarios

|                           | Scenario 1 (Cesium)                             | Scenario 2 (Mapbox)                                                  | Scenario 3 (three.js)                                                |
| ------------------------- | ----------------------------------------------- |:-------------------------------------------------------------------- |:-------------------------------------------------------------------- |
| Network performance    | Good and reliable                               | Good and reliable                                                    | Depends on implementation, can't be guaranteed                       |
| Client performance        | Decent at best, generally mediocre              | Very good for 2.5D, questionable for 3D                              | Promising, but depends on the implementation                         |
| LoD Buildings support         | All LoDs supported                              | LoD1.x (extruded footprints), LoD2 possible to implement             | All LoDs supported                                                   |
| multi-LoD management      | Out-of-the-box                                  | Requires implementation                                              | Requires implementation                                              |
| Terrain support           | Supported                                       | Requires implementation, conflicts with existing 2D approach         | Requires implementation                                              |
| Development workload      | Low                                             | Medium                                                               | High                                                                 |
| Maintenance workload      | Low                                             | Medium                                                               | High                                                                 |
| Vendor lock-in            | Low                                             | Medium                                                               | None                                                                 |

## 5 Recommendations

To summarise and conclude we have three recommendations for the realisation of a webviewer for a nation-wide 3D citymodel.

1. Performance over (too) many features
  
    Succesfully implementing a web viewer for a nation-wide 3D citymodel has its challenges. We believe that, above all else, the viewer must have excellent performance and must be easy to use. Features should be implemented only when they do not decrease performance or when they do not cause too much confusion to the user. This is further underpinned by the novel nature of 3D geoinformation and the lower level of comfort users have using 3D technologies. If the performance and intuitiveness of the viewer are not good, people will quickly discard the viewer.


2. Hand-tailored solution over compromising with an existing viewer platform

    To us, building a platform with multiple data sources, efficient viewing and displaying the right amount information to the information is always a process that requires heavy development and a lot of experimentation on the UI perspective. Therefore, developing a custom made solution using three.js and something like 3D Tiles (scenario 3) would, eventually, not have as much as difference in time spent than one building on top of CesiumJS. In fact, maintaining a clean and hand-tailored codebase could be an advantage of a from-scratch solution, than building and relying on external libraries and services.
    
    Therefore, while the easiest and more straightforward approach might seem to be an implementation on top of CesiumJS, we would strongly suggest that a more streamlined solution, tailored for the specific use case is developed. We do acknowledge that this also depends on the amount of time one is prepared to invest in development and maintenance.
    
    Depending on the available workforce, such a solution should be possible to be implemented in a range of a few months. We estimate that a single developer working full-time must be able to implement a working prototype in about 1-2 months, and a beta version stable enough to be published for public testing in 4-6 months.

3. Tight integration between viewer and download service and other related datasets

    While the viewer itself best served by a for visualisation optimised static copy (ie 3D Tiles) of the city model, we also believe that the city objects should be directly retrievable through the viewer in a data format that is good for dissemination. CityJSON must be offered here, but other formats could also be considered/offered. Ideally the user can directly select, query and download city objects through the viewer. This can be implemented using the OGC API Features service (any format can be used, since the format is not prescribed by the standard). The viewer (but also other applications) would then be able to directly communicate with this API to query city objects.
    
    The work to implement such a download functionality/service is independent of what scenario is chosen, since none of the discussed viewers support it out of the box.
    
    In addition, the viewer could be linked to other Feature API's that can deliver information from related dataset (The Dis-Geo idea: if an object is represented in multiple datasets that are supplementary to each other, one should be able to easily obtain all the information linked to that object). This would require that such Feature API services exist and that the respective data models are designed such that this linking is indeed easy to do (for instance using a shared identifier). This would be outside of the scope of the viewer development, but the viewer could indeed benefit from such an infrastructure.