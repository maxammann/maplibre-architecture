Each of the GL libraries uses different terminology in its public APIs. This is because each library attempts to feel natural to developers on that platform.

## Conflicts in usage

The following terms in the Mapbox Style Specification have different meanings on the native platforms. The table below examines these terms along with a limited selection of terms from the GL libraries’ public APIs. Definitions in _italics_ denote concepts that the Mapbox GL developers have no control over.

| Term | Web | iOS/macOS | Android |
| --- | --- | --- | --- |
| annotation | Directions API metadata | a point or shape overlay | _Java language feature_; Directions API metadata |
| attribute | part of an HTML element | vector tile metadata; Directions API metadata | — |
| bindings | Node something-something | _reactive programming built into Objective-C/Swift_ | _reactive programming in Java?_ |
| class | a way to toggle paint properties | _language feature_ | _language feature_ |
| filter | a way to hide features in a layer | _Apple compositing feature_ | a way to hide features in a layer |
| function | _JavaScript language feature_; a variable property value | _2 different Objective-C/Swift language features_ | _Java language feature_; a variable property value
| id | unique identifier | _Objective-C language keyword_ | unique identifier |
| layer | similar to an overlay, but part of the map | _Apple animation feature_ | similar to an overlay, but part of the map |
| map view | an occasion a map is viewed | a container for a map | a container for a map |
| region | — | akin to bounding box | bundle of resources for offline usage |
| plugin | _browser feature_; GL helper library | GL helper library; _obscure application extension feature_ (macOS) | GL helper library
| point | 1D shape | unit of distance; 1D shape | 1D shape |
| property | _JavaScript language feature_; an option for a style layer or vector feature | _Objective-C/Swift language feature_ | _Java language feature_; an option for a style layer or vector feature |

## Platform-specific naming conventions

These differences in terminology have led to a number of differences in the libraries’ public APIs, as illustrated in the following table. 🛣 denotes a term used in a client of the Mapbox Directions API that is used together with a GL library (Mapbox GL Directions, MapboxDirections.swift, Mapbox Java SDK).

| Web | iOS/macOS | Android | Qt |
| --- | --- | --- | --- |
| pixel | point | pixel | pixel |
| — | pixel | — | — |
| symbol | — | — | — |
| — | annotation | marker, annotation | annotation |
| marker | annotation view | marker view | — |
| sprite | annotation image | marker icon | annotation icon |
| popup | callout, callout view; popover (macOS) | info window | — |
| geometry | shape | geometry annotation | shape |
| source | content source | source | data source |
| GeoJSON source | shape source | GeoJSON source | GeoJSON data source |
| raster source | raster tile source | raster source | raster data source |
| vector source | vector tile source | vector source | vector data source |
| — | computed shape source | custom source | — |
| layer | style layer | layer | layer, style layer |
| — | OpenGL style layer | custom layer | custom layer |
| property | attribute | property | property |
| id | identifier | id | id |
| class | style class | — | class |
| image | style image | image | image |
| SDF icon | template image | — | — |
| expression operator | expression function, predicate operator | expression operator | — |
| function | — | — | — |
| filter | predicate | filter | — |
| rendered features | visible features | rendered features | — |
| source features | features in source | source features | — |
| — | offline pack | offline region | — |
| — | offline region | offline region definition | — |
| — | offline storage | offline manager | offline cache database |
| attribution | attribution | attribution | copyrights |
| bearing | direction, heading | bearing | bearing |
| padding | edge insets, edge padding | content padding | margins |
| LngLat | coordinate | LatLng | coordinate |
| bounds, bbox | coordinate bounds, region | LatLng bounds | bounds |
| project/unproject | convert | project/unproject | — |
| camera options | map camera | camera | — |
| — | user location annotation view, user dot | my location view | — |
| event listener | observer | listener | signal |
| handler | gesture recognizer | — | — |
| — | swipe | swipe, fling | — |
| click | tap (iOS) / click (macOS) | click, touch | click |
| double-click | double-tap (iOS) / double-click (macOS) | double-touch | double-click |
| — | long tap (iOS) / press (macOS) | long click, long press | — |
| pan | pan, scroll | scroll | pan |
| scroll, zoom | pinch | pinch | scale |
| annotation 🛣 | attribute | annotation | — |