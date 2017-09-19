Each of the GL libraries uses different terminology in its public APIs. This is because each library attempts to feel natural to developers on that platform.

## Comparing meanings of style specification terms on each platform

The following terms in the Mapbox Style Specification and Mapbox GL JS API have different meanings on the native platforms. Definitions in _italics_ denote concepts that the Mapbox GL developers have no control over.

| Term | Web | iOS/macOS | Android |
| --- | --- | --- | --- |
| annotation | Directions API metadata | a point or shape overlay | _Java language feature_; Directions API metadata |
| attribute | part of an HTML element | vector tile metadata; Directions API metadata | — |
| bindings | Node something-something | _reactive programming built into Objective-C_ | _reactive programming in Java?_ |
| class | a way to toggle paint properties | _language feature_ | _language feature_ |
| filter | a way to hide features in a layer | _Apple compositing feature_ | a way to hide features in a layer |
| function | _JavaScript language feature_; a variable property value | _Objective-C language feature_ | _Java language feature_; a variable property value
| id | unique identifier | _Objective-C language keyword_ | unique identifier |
| layer | similar to an overlay, but part of the map | _Apple animation feature_ | similar to an overlay, but part of the map |
| map view | an occasion a map is viewed | a container for a map | a container for a map |
| region | — | akin to bounding box | bundle of resources for offline usage |
| point | 1D shape | unit of distance; 1D shape | 1D shape |
| property | _JavaScript language feature_; an option for a style layer or vector feature | _Objective-C/Swift language feature_ | _Java language feature_; an option for a style layer or vector feature |

## Comparing public API terminology across platforms

These differences in terminology have led to a number of differences in the libraries’ public APIs, as illustrated in the following table. 🛣 denotes a term used in a client of the Mapbox Directions API that is used together with a GL library (Mapbox GL Directions, MapboxDirections.swift, Mapbox Java Services).

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
| layer | style layer | layer | layer, style layer |
| — | OpenGL style layer | custom layer | custom layer |
| property | attribute | property | property |
| id | identifier | id | id |
| class | style class | — | class |
| icon | style image | image | image |
| SDF icon | template image | — | — |
| function | style function | function | — |
| zoom function | camera style function | camera function | — |
| property function | source style function | source function | — |
| zoom-and-property function | composite style function | composite function | — |
| filter | predicate | filter | — |
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