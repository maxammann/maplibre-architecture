This document describes the required changes needed to update your code base from a `6.x.x` version of the Maps SDK for Android to a `7.x.x` version.

## New Mapbox default styles

The default Mapbox style URLs in the `Style` class have been bumped to their new versions. These visually improved map styles also bring changes to the names of layers. If your project queries for a certain map layer ID, make sure the layer is still present in the new version of `Mapbox Streets`, `Mapbox Light`, etc. [Logging each map style name with a `for` loop once the map has been loaded](https://www.mapbox.com/android-docs/maps/overview/styling-map/#retrieving-a-map-layer), is a great way to to view the names of the style’s layers. You can also use Mapbox Studio in a non-mobile browser to view the z-index order and the names of the style’s layers.

## Style object

`7.0.0` of the Maps SDK introduces the `Style` class. A `Style` object is a representation of the actively shown style on a map. Using the `Style`  class provides more flexibility to create and adjust styles. The relationship between the map and the underlying style, becomes clearer with a dedicated `Style` too.

A `Style`  object ***must*** be created and given to the map for the map to appear properly.  Create a `Style` by using a:

- default Mapbox style URL via the `Style` class
- custom map style URL from a Mapbox account
- JSON string

Before `7.0.0`, the map would default to the Mapbox Streets style if no style URL was given to the Maps SDK. Beginning with `7.0.0`, there's no longer any defaulting to the Mapbox Streets style. If the style fails to load or an invalid style URL is set, the map will become blank. An error message will be logged in the Android logcat and the `MapView.OnDidFailLoadingMapListener` callback will be triggered.

Here’s the setup for a `Mapbox Streets` map if Java 8 lambdas aren’t available:

    mapView.getMapAsync(new OnMapReadyCallback() {
      @Override
      public void onMapReady(MapboxMap mapboxMap) {
    
        mapboxMap.setStyle(Style.MAPBOX_STREETS, new Style.OnStyleLoaded() {
          @Override
          public void onStyleLoaded(@NonNull Style style) {

            // Map is set up and the style has loaded. Now you can add
            // data or make other map adjustments



          }
        });
      }
    });

Here’s the setup for a `Mapbox Streets` map if Java 8 lambdas are available:

    mapView.getMapAsync(mapboxMap ->
      mapboxMap.setStyle(Style.MAPBOX_STREETS, style -> {

        // Map is set up and the style has loaded. Now you can add data or make other map adjustments


      })
    );

Use the `fromUrl()` method if you’d like to load a custom map style URL, rather than a default Mapbox style (e.g. Streets, Light, Satellite Streets, etc.):

Using a URL:
`mapboxMap.setStyle(new Style.Builder().fromUrl(UNIQUE_STYLE_URL));`

Using JSON:
`mapboxMap.setStyle(new Style.Builder().fromJson(UNIQUE_STYLE_JSON));`

Use a builder object to compose all components of a style. `Style.Builder()` provides additional style-related components such as sources, layers, images. Use the builder to configure the style transition options.


    mapboxMap.setStyle(new Style.Builder()
             .withLayer(
               new BackgroundLayer("bg")
                 .withProperties(backgroundColor(rgb(120, 161, 226)))
             )
             .withLayer(
               new SymbolLayer("symbols-layer", "symbols-source")
                 .withProperties(iconImage("test-icon"))
             )
             .withSource(
               new GeoJsonSource("symbols-source", testPoints)
             )
             .withImage("test-icon", markerImage)
    );

Get the map’s `Style` to add a source or layer at a later point in your project:


    mapboxMap.getStyle().addLayer(layerObject);


    mapboxMap.getStyle().addSource(sourceObject);


## XML attributes 

The `mapbox_styleUrl` and `mapbox_styleJson` XML attributes have been removed from [the list of useable XML attributes.](https://github.com/mapbox/mapbox-gl-native/blob/master/platform/android/MapboxGLAndroidSDK/src/main/res/values/attrs.xml)  As described above, set the map style URL or JSON inside of the `onMapReady()` callback.


## New LatLng & LatLngBounds wrapping

Beginning in `7.0.0`, `LatLng` and `LatLngBounds` longitude values are no longer wrapped from -180 to 180. In order to represent bounds which cross the International Date Line, create longitude pairs of 170 and 190 or -170 and -190.


## Listener API removal

A couple of releases before `7.0.0`, singular callback methods have been deprecated. Starting in `7.0.0`, we are removing all deprecated listeners from the Maps SDK for Android. You can now register multiple callbacks at the same time for specific events. Some example include registering multiple camera callbacks or map change events. [More about map events](https://www.mapbox.com/android-docs/maps/overview/events/).


## Dedicated map change event API

Instead of capturing and transmitting all map events to listeners, the `7.0.0` release introduces a callback for each map change event which we support. This limits the overhead caused by the construct as well make it more easy to integrate map change event listeners.


## `MapClickListener` booleans

The `OnMapClickListener` and `OnMapLongClickListener` interfaces now return a `boolean`. Add `return true` if the click should be consumed and not passed further to other listeners registered afterwards. Otherwise, add `return false`.


## MarkerView API removal

The `MarkerView` has been deprecated for a while and with `7.0.0`, it has been removed from the Maps SDK. The Maps SDK `MarkerView` renders and synchronizes an Android SDK View on top of a map. `MarkerView` functionality is now provided in the [Mapbox MarkerView Plugin for Android](https://www.mapbox.com/android-docs/plugins/overview/markerview/).

## Annotations API deprecation

Starting in `7.0.0`, the old concept of annotations is deprecated. Classes such as `Polygon`, `Polyline`, and `Marker` will not longer be maintained. This also means classes such as `PolygonOptions` and `PolylineOptions` should not be used. Lastly, this also means that methods such as `addPolygon()`, `addPolyline()`, or `addMarker()` should not be used.

The Maps SDK is now using a more powerful, low-level layer system for adding annotations. A `FillLayer` should be used to create a polygon, a `CircleLayer`  to create a circle, a `LineLayer` to create a polyline, and a `SymbolLayer` to create marker icons and text. To help interact with this annotation system, Mapbox now offers and recommends [the Mapbox Annotation Plugin for Android](https://www.mapbox.com/android-docs/plugins/overview/annotation/). The plugin obfuscates the boilerplate code which is needed to adjust various properties of map text, icons, lines, circles, and polygons.


## LocationComponent API

The Maps SDK now has a [`LocationComponent`](https://www.mapbox.com/android-docs/maps/overview/location-component/). It’s responsible for displaying and customizing a device’s location on a Mapbox map. In the past, [the Location Layer Plugin](https://www.mapbox.com/android-docs/plugins/overview/location-layer/) was used for this, but the plugin has now been deprecated. No further work is being put into the plugin and we strongly recommend that you use the `LocationComponent`. 

[See the 7.0.0.-compatible LocationComponent examples in the Mapbox Android Demo app](https://github.com/mapbox/mapbox-android-demo/tree/b5970d57e5dac225093ec5923ec92446f3e79146/MapboxAndroidDemo/src/main/java/com/mapbox/mapboxandroiddemo/examples/location) to see correct code for using the `LocationComponent` in `7.0.0`.

In order to avoid `java.lang.NullPointerException: Attempt to invoke virtual method 'boolean com.mapbox.mapboxsdk.maps.Style.isFullyLoaded()' on a null object reference`, [the provided style parameter in the LocationComponent#activate method](https://github.com/mapbox/mapbox-android-demo/blob/b5970d57e5dac225093ec5923ec92446f3e79146/MapboxAndroidDemo/src/main/java/com/mapbox/mapboxandroiddemo/examples/location/LocationComponentActivity.java#L67) has to be `@NonNull` and fully loaded. The best way is to [pass the style provided in the OnStyleLoaded callback](https://github.com/mapbox/mapbox-android-demo/blob/b5970d57e5dac225093ec5923ec92446f3e79146/MapboxAndroidDemo/src/main/java/com/mapbox/mapboxandroiddemo/examples/location/LocationComponentActivity.java#L54-L55).


## Java 7 

Starting in `7.0.0`, support for Java 8 language features is reverted in the Maps SDK. As result, you won’t be forced to enable Java 8 language features in your project. You can remove the following code block from the `android` section of your app’s `build.gradle` file:


    compileOptions {
         sourceCompatibility JavaVersion.VERSION_1_8
         targetCompatibility JavaVersion.VERSION_1_8
    }


## OfflineRegionDefinition

A new type of offline region was introduced several releases before `7.0.0`. This region is based on a specific geometry instead of a bounding box. Because we couldn’t change the interface definition in a minor Maps SDK release, this update was postponed until the `7.0.0` release. This update adds more common elements to the interface definition.


## Location services and liblocation `1.X.X`

[Read the latest information and up-to-date code snippets about Mapbox’s Android Core library](https://www.mapbox.com/android-docs/core/overview/).

## New plugin naming

[The Mapbox Plugins for Android](https://www.mapbox.com/android-docs/plugins/overview/) are heavily dependent on the major semantic versioning number of the Maps SDK. They either won't compile or hide runtime bugs when paired with a different major version of the Maps SDK. To make the transition between versions easier and more educated without the need to jump into changelogs and compare repositories, we used the `7.0.0` release to make a small but helpful tweak to the plugins’ dependency names. The plugins’ artifacts now have a suffix stating which major version of the Maps SDK the plugins are compatible with. You’ll notice below that `v7` has been added in front of each plugin’s version number:


    mapbox-android-plugin-annotation-v7:0.4.0
    mapbox-android-plugin-building-v7:0.5.0
    mapbox-android-plugin-localization-v7:0.7.0
    mapbox-android-plugin-markerview-v7:0.2.0
    mapbox-android-plugin-offline-v7:0.4.0
    mapbox-android-plugin-places-v7:0.7.0
    mapbox-android-plugin-traffic-v7:0.8.0



## Miscellaneous


- `colorToRgbaString` is no longer in the `PropertyFactory` class. It’s in [the new `ColorUtils` class](https://github.com/mapbox/mapbox-gl-native/blob/release-iowaska/platform/android/MapboxGLAndroidSDK/src/main/java/com/mapbox/mapboxsdk/utils/ColorUtils.java).

- The `STATE_STYLE_URL` constant is no longer available in the `MapboxConstants` class.

- The import package path for referencing the `Style` class has changed. It’s now `import com.mapbox.mapboxsdk.maps.Style` and no longer `import com.mapbox.mapboxsdk.constants.Style`.

- The `MapboxMap` class’ `removeSource()` and `removeLayer()` methods now return booleans. The booleans state whether the removal has been successful. If you’d like to hold on to the reference, fetch it beforehand with respective `get` methods.

- Map click listeners can now consume the click event and stop it from being passed to listeners down the list (registered after the current one).

- Support for the `ZoomButtonsController` was removed.



