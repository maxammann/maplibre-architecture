This document describes the required changes needed to update your codebase using v5 of the SDK to the 6.0.0 version.

### Java 8:

The 6.0.0 version of the Mapbox Maps SDK for Android introduces the use of Java 8. To fix any Java versioning issues, ensure that you are using Gradle version of `3.0` or greater. Once you’ve done that, add the following `compileOptions`  to the `android` section of your app-level `build.gradle` file like so:


    android {
      ...
      compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
      }
    }

This can also be done via your project settings (File > Project Structure > Your_Module > Source Compatibility / Target Compatibility). This is no longer required with Android Studio 3.1.0, as the new dex compiler [D8](https://android-developers.googleblog.com/2017/08/next-generation-dex-compiler-now-in.html) will be enabled by default.


### Expressions vs functions / filters:

The 6.0.0 release changes the syntax used to perform apply data driven styling of the map. Functions and filters have been replaced by [expressions](https://www.mapbox.com/mapbox-gl-js/style-spec/#expressions). Any [layout property](https://www.mapbox.com/mapbox-gl-js/style-spec/#layout-property), [paint property](https://www.mapbox.com/mapbox-gl-js/style-spec/#paint-property), or [filter](https://www.mapbox.com/mapbox-gl-js/style-spec/#layer-filter) can now be specified as an *expression*. An expression defines a formula for computing the value of the property using the *operators* described below. The set of expression operators provided by Mapbox GL includes:

- Mathematical operators for performing arithmetic and other operations on numeric values
- Logical operators for manipulating boolean values and making conditional decisions
- String operators for manipulating strings
- Data operators, providing access to the properties of source features
- Camera operators, providing access to the parameters defining the current map view
### Expressions tips & tricks:
- Stops should be listed in ascending order
  - If you’ve got a list of stops where the `Object` value for each stop is a number, the stops must be in the order of their values from lowest to highest. In the example below, the two stops that affect the radius of the `CircleLayer`'s circles, are placed in the correct order because 2 is less than 180:
 
```
    circleRadius(
      interpolate(
        exponential(1.75f),
        zoom(),
        stop(12, 2),
        stop(22, 180)
    )),
```

- Wrap Android `int` colors with the color expression
  - [The](https://www.mapbox.com/mapbox-gl-js/style-spec/#expressions) [Mapbox](https://www.mapbox.com/mapbox-gl-js/style-spec/#expressions) [style specif](https://www.mapbox.com/mapbox-gl-js/style-spec/#expressions)[i](https://www.mapbox.com/mapbox-gl-js/style-spec/#expressions)[cation](https://www.mapbox.com/mapbox-gl-js/style-spec/#expressions) exposes the `rgb()` and `rgba()` color expressions. To convert an Android `int` color to a color expression, you need to wrap it with `color(int color)`. For example:  

```
stop(0, color(Color.parseColor("#F2F12D"))).
```

- When you don’t type your `get()` expressions explicitly, it is likely that the expression will implicitly *expect* them to be a certain type — and fail at runtime. They fall back to the style-spec default if they aren’t the expected type. To alter this behavior, use `toString()`, `toNumber()`, `toColor()`, or `toBoolean()` to explicitly *coerce* them to the desired type.

### Plugin support:
- Deprecation of setXListener
  - With the 6.0.0 release we are allowing listeners to be set through `addXListener` to support notifying multiple listeners for plugin support. Passing a `null` in the add method will listener not result in clearing any listeners. Please use equivalent `remove`XListener method instead.


### Location layer vs LocationView/trackingsettings:

With the 6.0.0 release, we are removing the deprecated LocationView and related tracking modes from the Maps SDK. These were deprecated in favor of [the Location](https://github.com/mapbox/mapbox-plugins-android/tree/master/plugin-locationlayer) [](https://github.com/mapbox/mapbox-plugins-android/tree/master/plugin-locationlayer)[Layer](https://github.com/mapbox/mapbox-plugins-android/tree/master/plugin-locationlayer) [](https://github.com/mapbox/mapbox-plugins-android/tree/master/plugin-locationlayer)[Plugin in](https://github.com/mapbox/mapbox-plugins-android/tree/master/plugin-locationlayer) [th](https://github.com/mapbox/mapbox-plugins-android/tree/master/plugin-locationlayer)e [plugins repository](https://github.com/mapbox/mapbox-plugins-android/tree/master/plugin-locationlayer). The plugin directly renders the user’s location in GL instead of relying on a synchronized Android SDK View. The 0.5.0 version of the Location Layer Plugin is compatible with the 6.0.0 release of the Maps SDK.


### `LatLngBounds`:
- The latitude values should be within a range of -90 to 90 when a  `LatLngBounds` object is created.
- LatLngBounds.Builder will create bounds with the shortest possible span.

```
     // The following bounds will be crossing the antimeridian (new date line)
     LatLngBounds latLngBounds = new LatLngBounds.Builder()
      .include(new LatLng(10, -170))
      .include(new LatLng(-10, 170))
      .build();
    
      // latlngBounds.getSpan() will be 20, 20  
    
      // The same could be done with
      latLngBounds = LatLngBounds.from(10, -170, -10, 170); 
    
```

Note that if you need bounds created out of the same points but spanning around different part of the earth (forming a much wider bounds)  you can use:

```
    LatLngBounds latLngBounds = LatLngBounds.from(10, 170, -10, -170);
    // latlngBounds.getSpan() will be 20, 340  
```

- The order of parameters for `LatLngBounds.union()` has changed. The parameter order for the `union()` method has been made consistent with other methods. That is: North, East, South, West.

### GeoJSON changes:
- GeoJSON objects are immutable
- All `setCoordinates()` methods are deprecated
- Geometry instantiation methods. 

The `fromCoordinates()` method on classes which implement  `Geometry`, were renamed to `fromLngLat()`. The method name for the `Point`  class is `fromLngLat()`.

The `Point`, `MultiPoint`, `LineString`, `MultiLineString`, `Polygon`, and `MultiPolygon` classes can no longer can be instantiated from an array of `double`. A  `List<Point>` should be used instead.

Pre-6.0.0 way:

    // release 5.x.x:
     Polygon domTower = Polygon.fromLngLats(new double[][][] {
              new double[][] {
                  new double[] {
                      5.12112557888031,
                      52.09071040847704
                      },
                  new double[] {
                      5.121227502822875,
                      52.09053901776669
                      },
                  new double[] {
                      5.121484994888306,
                      52.090601641371805
                      },
                  new double[] {
                      5.1213884353637695,
                      52.090766439912635
                      },
                  new double[] {
                      5.12112557888031,
                      52.09071040847704
                      }
                  }
              });

6.0.0+ way:

      // release 6.0.0
      List<List<Point>> lngLats = Arrays.asList(
        Arrays.asList(
          Point.fromLngLat(5.12112557888031, 52.09071040847704),
          Point.fromLngLat(5.121227502822875, 52.09053901776669),
          Point.fromLngLat(5.121484994888306, 52.090601641371805),
          Point.fromLngLat(5.1213884353637695, 52.090766439912635),
          Point.fromLngLat(5.12112557888031, 52.09071040847704)
        )
    );
    Polygon domTower = Polygon.fromLngLats(lngLats);


- Changes to getter methods’ names:

`Feature` and `FeatureCollection` getter methods don’t start with the word `get` any more. For example:

  - `Feature.getId()` → `Feature.id()`
  - `Feature.getProperty()` → `Feature.property()`
  - `FeatureCollection.getFeatures()` → `FeatureCollection.features()`, etc


- The `Position` class has been removed from the API.  The `Point` model class was simplified. A `Point` no longer holds a `Point`’s coordinates in a `Position` object.

Before the 6.0.0 release:

      Position position = Position.fromCoordinates(10, 10);
      Point point = Point.fromCoordinates(position);
      Point anotherPoint = Poin.fromCoordinates(new double[]{10d, 10d});
      double latitude = point.getCoordinates().getLatitude();
      double longitude = point.getCoordinates().getLongitude();

With the 6.0.0 release, you should do:

    Point point = Point.fromLngLat(10, 10);
    double latitude = point.getLatitude(); // no longer need to getCoordinates() first
    double longitude = point.getLongitude();


### Plugin support:
- Deprecation of setXListener
  - With the 6.0.0 release we are allowing listeners to be set through `addXListener` to support notifying multiple listeners for plugin support. Passing a `null` in the add method will listener not result in clearing any listeners. Please use equivalent `remove`XListener method instead.

