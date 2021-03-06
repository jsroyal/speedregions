# SpeedRegions
SpeedRegions is a work-in-progress utility library designed for use with [Graphhopper](https://github.com/graphhopper/graphhopper)
which allows geographic regions with different speed profiles (e.g. a city or a country) to be defined. 
SpeedRegions uses several types of text files containing data in [JSON](https://en.wikipedia.org/wiki/JSON) format as its input.
The most important text file type is an *UncompiledSpeedRules* file. Here's an example file:

	{
	  "rules" : [ {
		"multiplier" : 0.7,
		"matchRule" : {
		  "flagEncoders" : [ "car" ],
		  "regionTypes" : [ "valleta" ]
		}
	  } ],
	  "geoJson" : {
		"type" : "FeatureCollection",
		"features" : [ {
		  "type" : "Feature",
		  "properties" : {
			"regiontype" : "valleta"
		  },
		  "geometry" : {
			"type" : "Polygon",
			"coordinates" : [ [ [ 14.457384575, 35.89565677 ], [ 14.45653875, 35.899281932 ], [ 14.458598686, 35.902341063 ], [ 14.460328621, 35.903588517 ], [ 14.462642955, 35.903662185 ], [ 14.463669377, 35.903031668 ], [ 14.466885473, 35.903436003 ], [ 14.474620438, 35.900815895 ], [ 14.47560616, 35.902646356 ], [ 14.478262151, 35.903359408 ], [ 14.48170059, 35.903080892 ], [ 14.482825259, 35.905273952 ], [ 14.484661024, 35.906453246 ], [ 14.482064857, 35.909298792 ], [ 14.48261423, 35.911169136 ], [ 14.484530453, 35.912303757 ], [ 14.492025801, 35.914038195 ], [ 14.492099908, 35.916840502 ], [ 14.494218522, 35.918159473 ], [ 14.496863088, 35.917848707 ], [ 14.504244527, 35.914234066 ], [ 14.505821489, 35.912450829 ], [ 14.50531176, 35.910296543 ], [ 14.503015911, 35.90910952 ], [ 14.504048385, 35.908181328 ], [ 14.503548915, 35.905793336 ], [ 14.500927327, 35.904628272 ], [ 14.495495, 35.904402622 ], [ 14.494694111, 35.903789943 ], [ 14.498669763, 35.902729607 ], [ 14.499548669, 35.900796901 ], [ 14.498545044, 35.898904423 ], [ 14.497541163, 35.898554493 ], [ 14.502196889, 35.896830726 ], [ 14.504298724, 35.898354127 ], [ 14.506708899, 35.898981673 ], [ 14.508739395, 35.900704787 ], [ 14.512138017, 35.901597455 ], [ 14.513991283, 35.903362667 ], [ 14.516214595, 35.903592849 ], [ 14.520334468, 35.902897613 ], [ 14.522506008, 35.90169914 ], [ 14.522995309, 35.899607856 ], [ 14.521515746, 35.89784879 ], [ 14.5195844, 35.897552275 ], [ 14.519862388, 35.894473296 ], [ 14.518251389, 35.893098567 ], [ 14.515437882, 35.89195893 ], [ 14.514770674, 35.890967969 ], [ 14.515983301, 35.889085031 ], [ 14.516154084, 35.88682177 ], [ 14.515175118, 35.884706714 ], [ 14.516435314, 35.88291993 ], [ 14.516802508, 35.88092412 ], [ 14.515407889, 35.879252265 ], [ 14.51298533, 35.878784166 ], [ 14.50765095, 35.879475078 ], [ 14.505195661, 35.878075059 ], [ 14.502811432, 35.878618985 ], [ 14.500843023, 35.879815169 ], [ 14.498347497, 35.878257795 ], [ 14.49584433, 35.878623366 ], [ 14.494338579, 35.880284308 ], [ 14.494603199, 35.881985166 ], [ 14.484047356, 35.882519382 ], [ 14.481711218, 35.883442279 ], [ 14.481073397, 35.884899684 ], [ 14.478373636, 35.884188849 ], [ 14.470820536, 35.88460609 ], [ 14.468694102, 35.885361006 ], [ 14.46782897, 35.887694007 ], [ 14.463597861, 35.886755088 ], [ 14.461721805, 35.887686648 ], [ 14.458288577, 35.891024366 ], [ 14.457494404, 35.892720693 ], [ 14.457384575, 35.89565677 ] ] ]
		  }
		} ]
	  }
	}

This file has two sections - (1) the array called *rules*, which contains a single speed rule and 
(2) the object geoJson, which must always be a [geoJson feature collection](http://geojson.org/geojson-spec.html#feature-collection-objects).
This feature collection contains a single feature which defines a region around the city of Valleta, in Malta. 
The single speed rule applies to the Graphhopper car flag encoder (i.e. the speed profile for cars)
and slows down speeds by 30% (multiplier = 0.7) within Valleta. 
The *regionType* field links the Valleta geoJson feature with the speed rule - so different speed rules can
be applied to the same geographic region (for example, different speed rules for a truck or motorbike).

## Compiled vs uncompiled files
An uncompiled file is called 'uncompiled' because the SpeedRegions library still needs to build a spatial lookup
(a kind of map) which stores the geographic regions in a data structure optimised for querying the region a road
sits within. 
Building this spatial lookup can be slow and take a similar amount of time to building the graph - e.g. a couple of minutes just for the United Kingdom dependent on accuracy.
As speed rules will change often (e.g. make a road type a little faster or slower) but the geographic regions won't
change as often, we support pre-compiling this spatial tree before using it in Graphhopper. 
You can therefore compile the spatial tree once and then quickly modify the speed rules in the compiled file without
having to recompile the tree. 

## Types of JSON text file
When building the Graphhopper graph, you can give SpeedRegions one of two different types of JSON text files:

1. **UncompiledSpeedRules**. A JSON text file containing the speed rules and GeoJSON feature collection.
1. **CompiledSpeedRules**. A JSON text file containing the speed rules and the built spatial tree.

To help manipulate these files, we have a command line tools project. 
The command line tools loads and saves the following JSON file types:

1. **FeatureCollection**. A GeoJSON text file containing a single feature collection object.
1. **CompiledTree**. A JSON text file the containing built spatial tree.

You can use the command line tools to build the *CompiledTree* file from a *FeatureCollection* file. 
You build this tree once (and once only), then place it into a *CompiledSpeedRules* file. 
You can then tweak the speed rules in the *CompiledSpeedRules* as you like, without rebuilding the tree again.
Alternatively if you don't mind the extra time taken to rebuild the spatial tree when you're building the Graphhopper
graph, you can just use the *UncompiledSpeedRules* file as input without bothering with the other file types.

This is an example *CompiledSpeedRules* file:

	{
	  "rules" : [ {
		"speedsByRoadType" : { },
		"multiplier" : 0.7,
		"speedUnit" : "KM_PER_HOUR",
		"matchRule" : {
		  "flagEncoders" : [ "car" ],
		  "regionTypes" : [ "valleta" ]
		}
	  } ],
	  "tree" : {
		"bounds" : {
		  "minLng" : 14.4140625,
		  "maxLng" : 14.58984375,
		  "minLat" : 35.859375,
		  "maxLat" : 35.947265625
		},
		"regionType" : "valleta",
		"assignedPriority" : 1,
		"children" : [ ]
	  }
	}

In this case the tree is very small - it contains only one node. 
Generally speaking the tree will be a very large multi-level data structure with thousands or more nodes.

This is an example *FeatureCollection* file 
- it is pure geoJson and corresponds to the *geoJson* field in the *UncompiledSpeedRules* file:

	{
		"type" : "FeatureCollection",
		"features" : [ {
		  "type" : "Feature",
		  "properties" : {
			"regiontype" : "valleta"
		  },
		  "geometry" : {
			"type" : "Polygon",
			"coordinates" : [ [ [ 14.457384575, 35.89565677 ], [ 14.45653875, 35.899281932 ], [ 14.458598686, 35.902341063 ], [ 14.460328621, 35.903588517 ], [ 14.462642955, 35.903662185 ], [ 14.463669377, 35.903031668 ], [ 14.466885473, 35.903436003 ], [ 14.474620438, 35.900815895 ], [ 14.47560616, 35.902646356 ], [ 14.478262151, 35.903359408 ], [ 14.48170059, 35.903080892 ], [ 14.482825259, 35.905273952 ], [ 14.484661024, 35.906453246 ], [ 14.482064857, 35.909298792 ], [ 14.48261423, 35.911169136 ], [ 14.484530453, 35.912303757 ], [ 14.492025801, 35.914038195 ], [ 14.492099908, 35.916840502 ], [ 14.494218522, 35.918159473 ], [ 14.496863088, 35.917848707 ], [ 14.504244527, 35.914234066 ], [ 14.505821489, 35.912450829 ], [ 14.50531176, 35.910296543 ], [ 14.503015911, 35.90910952 ], [ 14.504048385, 35.908181328 ], [ 14.503548915, 35.905793336 ], [ 14.500927327, 35.904628272 ], [ 14.495495, 35.904402622 ], [ 14.494694111, 35.903789943 ], [ 14.498669763, 35.902729607 ], [ 14.499548669, 35.900796901 ], [ 14.498545044, 35.898904423 ], [ 14.497541163, 35.898554493 ], [ 14.502196889, 35.896830726 ], [ 14.504298724, 35.898354127 ], [ 14.506708899, 35.898981673 ], [ 14.508739395, 35.900704787 ], [ 14.512138017, 35.901597455 ], [ 14.513991283, 35.903362667 ], [ 14.516214595, 35.903592849 ], [ 14.520334468, 35.902897613 ], [ 14.522506008, 35.90169914 ], [ 14.522995309, 35.899607856 ], [ 14.521515746, 35.89784879 ], [ 14.5195844, 35.897552275 ], [ 14.519862388, 35.894473296 ], [ 14.518251389, 35.893098567 ], [ 14.515437882, 35.89195893 ], [ 14.514770674, 35.890967969 ], [ 14.515983301, 35.889085031 ], [ 14.516154084, 35.88682177 ], [ 14.515175118, 35.884706714 ], [ 14.516435314, 35.88291993 ], [ 14.516802508, 35.88092412 ], [ 14.515407889, 35.879252265 ], [ 14.51298533, 35.878784166 ], [ 14.50765095, 35.879475078 ], [ 14.505195661, 35.878075059 ], [ 14.502811432, 35.878618985 ], [ 14.500843023, 35.879815169 ], [ 14.498347497, 35.878257795 ], [ 14.49584433, 35.878623366 ], [ 14.494338579, 35.880284308 ], [ 14.494603199, 35.881985166 ], [ 14.484047356, 35.882519382 ], [ 14.481711218, 35.883442279 ], [ 14.481073397, 35.884899684 ], [ 14.478373636, 35.884188849 ], [ 14.470820536, 35.88460609 ], [ 14.468694102, 35.885361006 ], [ 14.46782897, 35.887694007 ], [ 14.463597861, 35.886755088 ], [ 14.461721805, 35.887686648 ], [ 14.458288577, 35.891024366 ], [ 14.457494404, 35.892720693 ], [ 14.457384575, 35.89565677 ] ] ]
		  }
		} ]
	}

This is our example *CompiledTree* file, containing only a single node; it corresponds to the *tree* field in the *CompiledSpeedRules* file:

	{
		"bounds" : {
		  "minLng" : 14.4140625,
		  "maxLng" : 14.58984375,
		  "minLat" : 35.859375,
		  "maxLat" : 35.947265625
		},
		"regionType" : "valleta",
		"assignedPriority" : 1,
		"children" : [ ]
	}

## Configuration of speed rules data
You can have as many speed rules as you like in the rules array.
Each speed rule can define a speed for a road type and a speed multiplier which is applied to a road type when either 
(a) no speed is specified in the rule for the road type or 
(b) the OSM *maxspeed* tag was present on the road edge and was used instead of the default speed for the road type.
The speeds for road types can be set in MILES_PER_HOUR or KM_PER_HOUR depending on the value of the *speedUnit* field.

Speed rules can have a parent-child hierarchy, where you specify a default set of speeds for an area,
for example a country, and give the rule containing these speeds an id.
Child rules can inherit the default speeds and multiplier from their parent rule if you set their parentId field,
and then override the speeds as needed. 
A child rule multiplies its parent's speeds by its *multiplier* field value.

The following *CompiledSpeedRules* example sets up speeds in MILES_PER_HOUR for different types
of roads in the UK, following the OpenStreetMap [highway tag conventions](http://wiki.openstreetmap.org/wiki/Key:highway). 
A rule is setup with id 'TypicalUKSpeeds' to apply to the regionType 'UK', 
which would correspond to a region covering the whole UK. 
Different rules for regionTypes London and Birmingham are then setup with a parentId of TypicalUKSpeeds 
and a multiplier of between 0.6-0.8. 
The rules for London and Birmingham will therefore use the speeds setup in their parent rule record (the UK record), 
but reduced by between 60% and 80%.

To specify speed in kilometres instead set the *speedUnit* field to KM_PER_HOUR.

	{
	  "rules" : [
		{
			"parentId" : "TypicalUKSpeeds",		
			"multiplier" : 0.6,
			"matchRule" : {
			  "flagEncoders" : [ "car" ],
			  "regionTypes" : [ "InnerLondon" ]
			}
		}		
		,{
			"parentId" : "TypicalUKSpeeds",		
			"multiplier" : 0.8,
			"matchRule" : {
			  "flagEncoders" : [ "car" ],
			  "regionTypes" : [ "OuterLondon" ]
			}
		}		
		,{
			"parentId" : "TypicalUKSpeeds",		
			"multiplier" : 0.8,
			"matchRule" : {
			  "flagEncoders" : [ "car" ],
			  "regionTypes" : [ "Birmingham" ]
			}
		}			
		,{
		  "id" : "TypicalUKSpeeds",
		  "speedsByRoadType" : {
			"living_street" : 3.1,
			"motorroad" : 55.9,
			"motorway" : 62.1,
			"motorway_link" : 43.5,
			"primary" : 40.4,
			"primary_link" : 37.3,
			"residential" : 18.6,
			"road" : 12.4,
			"secondary" : 37.3,
			"secondary_link" : 31.1,
			"service" : 12.4,
			"tertiary" : 31.1,
			"tertiary_link" : 24.9,
			"track" : 9.3,
			"trunk" : 43.5,
			"trunk_link" : 40.4,
			"unclassified" : 18.6
		  },
		  "multiplier" : 1.0,
		  "speedUnit" : "MILES_PER_HOUR",
		  "matchRule" : {
			"flagEncoders" : [ "car" ],
			"regionTypes" : [ "UK" ]
		  }
		}	
	  ],
	  "tree" : {
		....
	  }
	}

## Configuration of feature collection data
A featureCollection should only contain [Polygon](http://geojson.org/geojson-spec.html#polygon) or 
[MultiPolygon](http://geojson.org/geojson-spec.html#multipolygon) geometry types.
Each feature should have a **regionType** property which links to the speed rules.
RegionType could refer to a geographic area - e.g. London - or a type of geographic area (e.g. 'big city').
Multiple features can therefore have the same regionType, there is no requirement for it to be a unique value per feature.

The order of a feature in the featureCollection is used as its priority 
when assigning a geographic area (technically a leaf node in the spatial tree) to a regionType.
If for example, you have a featureCollection containing polygons boundaries for United Kingdom, 
Central London and Outer London, your first feature should be the smallest or innermost feature - Central London - 
followed by Outer London (which Central London sits within) and then the United Kingdom. 
The order within the collection is therefore used to model overlapping polygons where one polygon sits within another.
	
## Building a Graphhopper graph with speed regions
The integration with Graphhopper is currently experimental and available for car speed profile only.
Full integration within the Graphhopper project is planned.
See the projects com.opendoorlogistics.speedregions.experimental.gh0.5 and com.opendoorlogistics.speedregions.experimental.ghlatest
to build a car profile graph using speed regions with Graphhopper 0.5 and Graphhopper latest release (version 0.7 as of 28/9/2016).

To build a Graphhopper graph using speed regions, build the projects using Maven and run from the command line as shown [here](http://www.opendoorlogistics.com/tutorials/tutorial-vi-advance-configuration/building-road-network-graphs/)
but with one of two sets of additional command line arguments:

* Using a *CompiledSpeedRules* file. Add the argument:

		speedregions.compiled=your_compiled_speed_rules_filename

* Using an *UncompiledSpeedRules* file. Add the two arguments:

		speedregions.uncompiled=your_uncompiled_speed_rules_filename
		speedregions.tolerance=tolerance_in_metres
	
### Choosing the speed regions tolerance value
The tolerance measures how accurate the spatial tree will be.
Technically speaking, the spatial tree divides the world up into rectangles, assigning each rectangle to a single region.
The tolerance is the minimum side length of these rectangles; so no rectangle will be created with a side length less
than *tolerance_in_metres*.

Road edges whose centre is within *tolerance_in_metres* of a region boundary may therefore be placed in the wrong region.
Generally speaking, the effect of placing an edge already near a region boundary within the wrong region 
will be small, so tolerance can be a relatively high value.
The tolerance value should however be signficantly smaller than the approximate width or height of your regions,
or your regions will not be represented properly or not even used.
The building of the spatial tree (i.e. the compilation step) takes a lot longer the smaller the tolerance is
and the compiled files will be a lot larger.

We recommend using a starting value of 500m and only reducing this if needed.
	
## Visualising regions and the tree in ODL Studio
ODL Studio can be used for visualising the tree (TODO DOCUMENT THIS).

![Speed regions shown for London built with 100m tolerance and a route superimposed with and without the regions.](http://www.opendoorlogistics.com/wp-content/uploads/speedregions/London-route-comparison.png)

## Future developments

* Full integration into Graphhopper project.

* Binary format for compiled tree - the large size of the compile tree in the CompiledSpeedRules file 
	can limit the practicallity of speed regions to national level. 
	The file size could be reduced dramatically with a small amount of future development work 
	(for example, the compressed file size of the compiled JSON is just ~3% of the uncompressed).
	
* Integrate speeds based on *trackType* tag, currently we ignore these.