# Mapbox GL Collision Detection
This document is meant to capture the overall architecture of symbol collision detection at the end of 2017, after making the change to global (e.g. cross-source) viewport-based collision detection.

Mapbox vector tiles contain a collection of features to draw on a map, along with labels (either text or icons, generically called “symbols”) to attach to those features. Generally, the set of labels in a tile is larger than the set of labels that can be displayed on the screen at one time without overlap. To preserve legibility, we automatically choose subsets of those labels to display on the screen for any given frame: this is what we call “collision detection”. The set of labels that we choose can change dramatically as camera parameters change (pitch/rotation/zoom/etc.) because in general the shape of text itself will not be affected the same way as the underlying geometry the text is attached to.

**Collision Detection Desiderata**

- Correct: Labels don’t overlap!
- Prioritized: More important labels are chosen before less important labels
- Deterministic: viewing the same map data from the same camera perspective will always generate the same results
- Stable: During camera operations such as panning/zooming/rotating, labels should not disappear unless there’s no longer space for them on the screen
- Global: collision detection should account for all labels on the screen, no matter their source
- Smooth: when collisions happen, we should be able to animate fade transitions to remove “collided” labels
- Fast: collision detection should be integrated into a 60 FPS rendering loop, which means synchronous calculations that take more than a few milliseconds are unacceptable. On the other hand, latency of a couple hundred milliseconds in actually detecting a new collision is perceptually acceptable.


## Basic Strategy

The core of our collision detection algorithm uses a two-dimensional spatial data structure we call a `GridIndex` that is optimized for fast insertion and fast querying of “bounding geometries” for symbols on the map. Every time we run collision detection, we go through the set of all items we’d like to display, in order of their importance:

1. We use information about the size/style/position of the label, along with the current camera position, to calculate a bounding geometry for the symbol as it will display on the current screen
2. We check if the bounding geometry for the symbol fits in unused space in the `GridIndex`.
3. If there’s room, we insert the geometry, thereby marking the space as “used”. Otherwise, we mark this label “collided” and move on to the next label

The “global”, “smooth”, and “fast” desiderata add an important challenge for Mapbox GL because Mapbox maps are frequently combining data from multiple sources, and any change in any of the sources can potentially affect symbols from all of the other sources (because of “cascade” effects, introducing a symbol B that blocks symbol A could open up room for symbol C to display, which could in turn block a previously-showing symbol D…). To manage this challenge, we use a data structure we call the `CrossTileSymbolIndex` which allows us to globally synchronize collision detection (and symbol fade animation).


## `GridIndex`

The `GridIndex` is a two-dimensional spatial data structure. Although the data structure itself is context-agnostic, in practice we use a `GridIndex` in which the 2D plane is the “viewport” — that is, the plane of the device’s screen. This is in contrast to the “tile” or “map” plane used to represent our underlying data. While most map-drawing logic is easiest to process in “map” coordinates, “viewport” coordinates are a better fit for text because in general we prefer text that is aligned flat relative to the screen.

`GridIndex` holds two types of geometries:

- Rectangles: used to represent “point” labels (e.g. a city name)
- Circles: used to represent “line” labels (e.g. a street name that follows the curve of the street)

We limit ourselves to these two geometry types because it easy to quickly test for the three possible types of intersection (rectangle-rectangle, circle-circle, and rectangle-circle).

**Rectangles**
Rectangles are a fairly good fit for point labels because:

- Text in point labels is laid out in a straight line
- Point labels are *generally* oriented to the viewport, which means their orientation won’t change during map rotation (although their position will change)
- Multi-line point labels are, as much as possible, balanced so that the lines approximate a rectangle: https://github.com/mapbox/hey/issues/6079
- Text is *generally* drawn on the viewport plane (i.e. flat relative to the screen). When it is drawn “pitched” (i.e. flat relative to the map), a viewport-aligned rectangle can still serve as a reasonable approximation, because the pitched projection of the text will be a trapezoid circumscribed by the rectangle.

**Circles**
Line labels are represented as a series of circles that follow the course of the underlying line geometry across the length of the label. The circles are chosen so that (1) both the beginning and end of the label will be covered by circles, (2) along the line, the gap between adjacent circles will never be too large, (3) for performance reasons, adjacent circles won’t significantly overlap. The code that chooses which circles to use for a label is tightly integrated with the line label rendering code, and it’s also code we’re thinking of changing: I’ll consider it out of scope for this document.

Representing a label as several circles is more expensive than representing it as a single rectangle, but still relatively cheap. Using multiple circles allows us to approximate arbitrary line geometries (and in fact there are outstanding feature requests that would require us to support other arbitrary collision geometries such as “don’t let labels get drawn over this lake” — we would probably implement support by converting those geometries into circles first).

The key benefit of circles relative to rectangles is that they are *stable under rotation*. Since line labels are attached to map geometry, they necessarily rotate with the map, unlike point labels, which are usually fixed to the viewport orientation. If we were using small rectangles to represent line labels (as we did before the most recent changes), then the *viewport-aligned* rectangles would rotate relative to the map during map rotation. The rotation change alone could cause two adjacent line labels to collide, violating our “stability” desideratum. Circles, on the other hand, are conveniently totally unaffected by rotation.

**The Grid**
The “grid” part of `GridIndex` is a technique for making queries faster that relies on the assumption that most geometries that are stored in the index will only occupy a relatively small portion of the plane. When we use the `GridIndex` to represent a (for example) 600px by 600px viewport, we split the 600x600px plane into a grid of 400 (20x20) 30px-square “cells”.

For each bounding geometry we insert, we check to see which cells the geometry intersects with, and make an entry in that cell representing a pointer to the bounding geometry. So for example a rectangle that spans from [x: 10, y: 10] to [x: 70, y: 20] would be inserted into the first three cells in the grid (which would span from [0,0] to [90, 30]).

When we want to test if a bounding geometry intersects with something already in the `GridIndex`, we first find the set of cells it intersects with. Then, for each of the cells it intersects with, we look up the set of bounding geometries contained in that cell, and directly test the geometries for intersection.

Given the assumption that bounding geometries are relatively uniformly distributed (which is a reasonable assumption for map labels), this approach has an attractive algorithmic property: doubling the dimensions of the index (and thus roughly doubling the number of entries) is not expected to have a significant impact on the cost of a query against the index. This is because for any given query the cost of finding the right cells is near constant, and the number of comparisons to run against each cell is a function of the *density* of the index, but not its overall size. Since our collision detection algorithm has to test every label for collision against every other label, near-constant-time queries allow us to keep collision detection time linear in the number of labels.

If the cell size is too small, the “fixed” part of a query (looking up cells) becomes too expensive, while if the cell size is too large, the index degenerates towards a case in which every geometry has to be compared against every other geometry on every query. We derived the 30px-square cell size by experimental profiling of common map animations. Conceptually, this is roughly the smallest size at which individual “collision circles” for 16px-high are *likely* to fit within a single cell and at most four cells. Optimizing for individual circles makes sense because the most expensive collision detection scenario for us is a dense network of road labels, which will be almost entirely represented with circles.

## `CrossTileSymbolIndex`

We satisfy the  “smooth” desideratum by using animations to fade symbols in (when they’re newly allowed to appear) and out (when they collide with another label). However, our tile-based map presents a challenge: symbols that are logically separate (e.g. “San Francisco” at zoom level 10 and “San Francisco” at z11) may be *conceptually* tied together, and the user expects consistent animation between the them (i.e. “San Francisco” should not fade out and fade back in when the z10 version is replaced by the z11 version, and if “San Francisco” is fading in while the zoom level is changing, the animation shouldn’t be interrupted by the zoom level change).

The `CrossTileSymbolIndex` solves this problem by providing a global index of all symbols, which identifies “duplicate” symbols across different zoom levels and assigns them a shared unique identifier called a `crossTileID`.

**Opacities**
Uploading data to the GPU is expensive, so to satisfy the “fast” desideratum, the only data we upload as a result of collision detection is a small “symbol opacity”, which is a “current opacity” between 0 and 1, and a “target opacity” of either 0 or 1.  The shader running on the GPU is responsible for interpolating between the current opacity and the target opacity using a clock parameter passed in as an argument.

The output of the collision detection algorithm is used to set the target opacity of every symbol to either 0 or 1. Every time the opacities are updated by collision detection, the “clock parameter” is essentially reset to zero, and the CPU re-calculates the baseline “current opacity”. One way to conceptualize this is “if the current opacity is different from the target opacity, a fade animation is in progress”.

Working with the shared, unique `crossTileID` makes it easy to handle smooth animations between zoom levels — instead of storing the current/target opacity for each symbol, we store it once per `crossTileID`. Then, at render time, we assign that opacity state to the “front-most” symbol, and assign the rest of the symbols the “invisible” opacity state [0,0].

Consider the “San Francisco” example, in which the label starts a 300ms fade-in while we are at zoom 10, and then we cross over into zoom 11 while the fade is still in progress.

1. “San Francisco” at z10 is given `crossTileID` 12345. It starts fading in, so is given opacity state [0,1].
2. z11 tile loads, “San Francisco” is given ID 12345, but at render time given opacity [0,0]
3. After 150 ms collision detection runs and opacities are updated. 12345 opacity is updated to [.5,1]
4. z10 tile is unloaded. At this point z11 version of “San Francisco” is the front-most symbol for 12345, so it is rendered with [.5,1], allowing the fade-in animation to continue
5. After another 150ms, the fade in animation completes (without requiring any opacity updates)
6. After another 150ms, collision detection runs again and 12345’s opacity is updated to the “finished” state of [1,1].

This opacity strategy also gives us a way to start showing the data from newly loaded tiles before we’ve had a chance to re-run the global collision detection algorithm with data from that tile: for symbols that share a cross-tile ID with an already existing symbol, we just pick up the pre-existing opacity state, while we assign all new cross-tile IDs an opacity state of [0,0]. The general effect while zooming in is that a tile will start showing with its most important POIs (the ones shared with the previous zoom level), and then some short period of time after the tile loads, the smaller POIs and road labels will start fading in.

**Detecting Duplicates**
The `CrossTileSymbolIndex` is a collection of `CrossTileSymbolLayerIndex`es: duplicates can only exist within a single layer. Each `CrossTileSymbolLayerIndex` contains, for each integer zoom level, a map of tile coordinates to a `TileLayerIndex`. A `TileLayerIndex` represents all the symbols for a given layer and tile. It has a `findMatches` method that can identify symbols in its tile/layer that are duplicate with symbols in another tile/layer. The query logic is, for each symbol:

1. If the query symbol is text, load the set of symbols that have the exact same text (often just one symbol). If the query symbol is an icon, load the set of all icons.
2. Convert the query symbol into “world coordinates” (see https://github.com/mapbox/hey/issues/6350) at the zoom level of the index. If the index is at z11 and the query symbol is at z10, the conversion essentially looks like a doubling of the query coordinates.
3. Reduce the precision of the coordinates — this is a type of rounding operation that allows us to detect “very close” coordinates as matches
4. For each local symbol, test if the scaled coordinates of the query are within a small “tolerance” limit of each other. If so, mark the query symbol as a duplicate by copying the `crossTileID` into it.

The index is updated every time a tile is added to the render tree. On adding a tile, the algorithm is:

1. Iterate over all children of the tile in the index, starting from the highest resolution zoom, and find any duplicates. Then, iterate over all parents of this tile, still in order of decreasing zoom. For each symbol in this tile, assign the first duplicate `crossTileID` you find.
2. For each symbol that didn’t get assigned a duplicate `crossTileID` create a new unique `crossTileID`
3. Remove index entries for any tiles that have been removed from the render tree.

A common operation is to replace a tile at one zoom level with a tile at a higher or lower zoom level. In many cases, the two tiles will both render simultaneously for some period before the old tile is removed, but it is also possible for there to be an instantaneous (i.e. single frame) swap between tiles in the render tree. In this case, an important property of the above algorithm is that the duplicate detection in step (1) happens *before* the unused tile pruning in step (3).

