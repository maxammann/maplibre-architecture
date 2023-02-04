# Mapbox GL Coordinate Systems

> **TL;DR:** Unsure how a point on a map gets transformed from a WGS84 latitude/longitude into x and y coordinates of a pixel on your screen? ~~Love matrix math~~ Willing to skim past a bunch of math? Read on… 
> 
> Just want to see some of the guts of Mapbox GL without any context? Check out the [coordinate transformation demo I made](https://chrisloer.github.io/mapbox-gl-coordinates)

For the last two months, I've been working on [making labels easier to read on pitched maps](https://github.com/mapbox/mapbox-gl-js/pull/4547), and as my first time really working with 3D graphics, it's been a crash course in trigonometry and linear algebra. The work brought back a surprising number of pleasant memories from high school trig class, and made me grateful for all the teachers (from Mr. Hatter twenty years ago to our local math teacher [@anandthakker](https://github.com/anandthakker) today) who have worked to prepare me for this job!

3D graphics are [all about transforming points between different coordinate systems](https://learnopengl.com/#!Getting-started/Coordinate-Systems), and as I followed our code to figure out how our transformations worked, I kept getting tripped up because I didn't realize which coordinate system the code was using. In this devlog, I'm trying to answer the question I didn't know I needed to ask when I started: what coordinate systems do we use in Mapbox GL to turn a bunch of points defined by latitude and longitude into a bunch of points with pixel x and y coordinates on your screen?

If you’re working directly on Mapbox GL code, this is important stuff to know: otherwise, these implementation details are for the most part mercifully hidden.

I'll start where @lyzidiamond's [awesome devlog on projections and coordinate systems](https://github.com/mapbox/hey/issues/6314) left off: WGS84 latitude and longitude.

## Where's the Washington Monument?

Let's follow how one point gets transformed through our different coordinate systems. The Washington Monument is located at approximately `(longitude: -77.035915, latitude: 38.889814)` in the WGS84 coordinate system. What happens if we ask Mapbox GL to draw a cat at just that point?

![google made this easy](./google-made-this-easy.jpg)

There are five coordinate systems we'll move through:


- WGS84
- World Coordinates
- Tile Coordinates
- GL Coordinates
- Screen Coordinates/NDC

**Projecting WGS84 to Web Mercator/"World Coordinates"**
As [@lyzidiamond](https://github.com/lyzidiamond) pointed out, computers love squares, so our first step is to take those spherical coordinates and turn them into coordinates on a rectangular 2D world (aka a "Pseudo-Mercator Projection"). Our particular projection imagines the world as a giant grid of pixels with the size:


    worldSize = tileSize * number of tiles across one dimension

We use a tile size of `512` (roughly: this means a tile is 512 pixels square, although that would only be true at the base zoom level and without any pitch). I'll assume we're rendering tiles at zoom level `11` (which means there are `2^11` tile positions across both dimensions). That gives us:


    worldSize = 512 * 2^11 = 1048576

Then getting an x coordinate is easy, we just shift by 180˚ (because we start with x=0 at -180˚) and multiply by the world size:


    x coord = (180˚ + longitude) / 360˚ * worldSize 
            = (180˚ + -77.035915˚) / 360˚ * 1048576
            = 299,904

For the y coordinate, we apply the [funky y stretch that is Web Mercator](https://en.wikipedia.org/wiki/Web_Mercator):


    y = ln(tan(45˚ + latitude / 2))
      = ln(tan(45˚ + 38.889814˚ / 2))
      = 0.73781742861
    y coord = (180˚ - y * (180 / π)) / 360˚ * worldSize
            = (180˚ - 42.27382˚) / 360˚ * 1048576
            = 401,156

OK, so `(x: 299904, y: 401156)` may be a great internal representation, but it's awkward and you have to know what zoom level it applies to. We can represent the same world coordinate by storing the zoom level and dividing the x and y components by the tile size. Our Washington Monument coordinate then becomes: `(585.7471, 783.5067, z11)`. One nice thing about storing coordinates this way is that it's really simple to transform them to different zoom levels.


![world coordinates](https://d2mxuefqeaa7sj.cloudfront.net/s_0240CF0C8C6EAE4CE3E50E3DFEF825E2238C6126BE7AB9B1C6F141A68979605B_1497564253787_Washington+Monument+World+Coordinates.png)


**Web Mercator -> Tile Coordinates**
Now we're making progress, because we have almost exactly what we need to ask the server for a tile that contains this point: we just drop the sub-integer precision and ask for `(585/783/11)`.

If we're drawing a vector tile, it will have geometry data for all the things on the map that we need to draw *around* our Washington Monument Cat. The [Mapbox vector tile spec](https://github.com/mapbox/vector-tile-spec/tree/master/2.1#41-layers) specifies that tiles should have a defined `extent` that they use internally for coordinates. For a tile with an `extent` of 1024, `(0,0)` would be the top left corner of the tile and `(1024,1024)` would be on the bottom right.

Whatever tile coordinates come in, Mapbox GL internally normalizes to an `extent` of 8192 — since our tiles are (kind of) 512 pixels wide, that means our tile coordinates have a precision of 512/8192 = 1/16th of a pixel. To represent our Washington Monument coordinates in tile coordinates, we just multiply the remainder by the extent, so:


    (x: 585.7471, y: 783.5067, zoom: 11)

Becomes:


    Tile: 585/783/11
    Tile Coordinates: (.7471 * 8192, .5067 * 8192) = (x: 6120, y: 4151)

![tile coordinates](https://d2mxuefqeaa7sj.cloudfront.net/s_0240CF0C8C6EAE4CE3E50E3DFEF825E2238C6126BE7AB9B1C6F141A68979605B_1497564323856_Washington+Monument+Tile+Coordinates.png)

**Tile Coordinates -> 4D GL Coordinates**
When Mapbox GL loads a map tile, it pulls together all the data for the tile and turns it into a representation the GPU can understand. In general, the representation we give to the GPU is specified in tile coordinates — the advantage here is that we only have to build the tile once, but then we can ask the GPU to quickly draw the same tile with different bearing, pitch, zoom, and pan parameters.

The code we have that runs on the GPU is called a "shader": it takes tile coordinate inputs and outputs four dimensional "GL Coordinates" in the form `(x, y, z, w)`. Wait, four dimensions? Don't worry, it's just for doing math.

`x` and `y` are pretty straightforward left/right and up/down. `z` is only slightly trickier: it represents position along an axis directly orthogonal to your screen. `w` is the more mysterious component, but it's necessary for generating the "perspective" effect: you can think of it as a "scaling" parameter for the other components, or as a measure of how far away they are from the viewer. When we draw labels, it's very useful to know the `w` component of a point, because it gives us a measure of how large the items around us will be (a higher `w` value means the perspective projection will make everything around our point smaller when it ultimately displays on the screen).

Converting from tile coordinates to GL coordinates involves applying a lot of "transformations" to the coordinates: one to apply perspective, one to move the imaginary camera away from the map, one to center the map, one to rotate the map, one to pitch the map, etc. Each one of these transformations can be represented as a 4D matrix, and all of the matrices can be combined into one matrix that applies all the transformations in one mathematical operation. These transformations build on top of some pixel-based parameters (size of the viewport, idealized size of tile) so we sometimes talk about the resulting coordinates as "pixel" coordinates although it's a fairly indirect connection. You might also see this coordinate system referred to as ["clip space"](https://en.wikipedia.org/wiki/Clip_coordinates).

Let's see what that looks like for a very specific view of our map. Note that each tile has its own projection matrix: before going into GL coordinates, we first have to go back from tile coordinates to world coordinates so that the points from different tiles all have the same frame of reference.


    Zoom level: 11.6
    Map Center: 38.891/-77.0822 (over Washington DC)
    Bearing: -23.2 degrees
    Pitch: 45 degrees
    Viewport dimensions: 862 by 742 pixels
    Tile: 585/783/11 (covering a portion of Washington DC)

Gives a transformation matrix of:

|   | x        | y        | z        | w        |
| - | -------- | -------- | -------- | -------- |
| x | 0.224    | -0.079   | -0.026   | -0.026   |
| y | -0.096   | -0.184   | -0.062   | -0.061   |
| z | 0.000    | 0.108    | -0.036   | -0.036   |
| w | -503.244 | 1071.633 | 1469.955 | 1470.211 |

There's way too much to unpack there in one devlog, but if you want to look at all the individual transformations to get a feel for how they fit together to make a 3D projection, and how they change as you move around the map, check out this [GL coordinate transformation demo I made](https://chrisloer.github.io/mapbox-gl-coordinates).

To get from tile coordinates to GL coordinates, we just turn the tile coordinates into a 4D vector with a z-value of 0 (ignoring buildings for now, sorry [@lbud](https://github.com/lbud)!) and "neutral" w-value of 1. We then [apply the transformation matrix](https://en.wikipedia.org/wiki/Transformation_matrix) to get our GL coordinate, so:


    (x: 6120, y: 4151, z: 0, w: 1)

Becomes:

|                     | x                        | y                        | z                        | w                       |
| ------------------- | ------------------------ | ------------------------ | ------------------------ | ----------------------- |
| input x = 6120      | 0.224 * 6120 = 1370.88   | -0.079 * 6120 = -483.48  | -0.026 * 6120 = -159.12  | -0.026 * 6120 = -159.12 |
| input y = 4151      | -0.096 * 4151 = -398.496 | -0.184 * 4151 = -763.784 | -0.062 * 4151 = -257.362 | -0.061 * 4151 = 253.311 |
| input z = 0         | 0.000 * 0 = 0            | 0.108 * 0 = 0            | -0.036 * 0 = 0           | -0.036 * 0 = 0          |
| input w = 1         | -503.244 * 1 = -503.244  | 1071.633 * 1 = 1071.633  | 1469.955 * 1 = 1469.955  | 1470.211 * 1 = 1470.211 |
| **Output (Summed)** | 469.14                   | -175.631                 | 1053.473                 | 1057.78                 |

😬 (yikes, the numbers don’t exactly match up with the machine output below! My excuse is that I did the math by hand at low precision…)


    (x: 472.1721, y: -177.8471, z: 1052.9670, w: 1053.7176)


![gl coordinates](https://d2mxuefqeaa7sj.cloudfront.net/s_0240CF0C8C6EAE4CE3E50E3DFEF825E2238C6126BE7AB9B1C6F141A68979605B_1497564431838_Washington+Monument+GL+Coordinates.png)


When you pan/rotate/tilt our map and it animates at a smooth 60 fps, what's happening is that we're changing the transformation matrix and then passing that modified matrix to the GPU, which does a massively parallel (and mind-bogglingly fast) application of that transformation matrix to all the points it needs to draw.

**4D GL Coordinates -> 2D Pixels on your screen**
Our shader code is done once it outputs GL Coordinates, but OpenGL still does some more work to turn the coordinates into pixel locations on your screen.

First, it transforms to "Normalized Device Coordinates" by dividing the `x, y, z` components of the GL Coordinate by the `w` component. The resulting values look like:


    x: -1,1 --> left to right of viewport
    y: -1,1 --> bottom to top of viewport
    z: -1,1 --> depth (not directly visible)

Any values outside of the range `[-1,1]` indicate points that are off the screen — they are (kind of) thrown away. You can see that another way to think of `w` is as "the bounds of what coordinates are visible at this distance".

In our example, the GL Coordinates:


         (x: 472.1721, y: -177.8471, z: 1052.9670, w: 1053.7176)

Become NDC:


            (472.1721 / 1053.72, -177.8471 / 1053.72, 1052.9670 / 1053.72)
         -> (x: 0.4481, y: -0.1688, z: 0.9993)

The z-value of NDC doesn't affect the 2D pixel coordinates. Instead, it's used for combining multiple layers (for instance, a bridge drawn over a river). The z-value tells the GPU which points are "in front" of other points: for opaque layers only the frontmost layer will show, but for translucent layers the z-order can be used for blending. Actually, I don't really know what I'm talking about here: ask [@lbud](https://github.com/lbud)!

Finally, OpenGL translates NDC to pixel coordinates based on the size of the viewport:

![NDC to pixel](https://d2mxuefqeaa7sj.cloudfront.net/s_0240CF0C8C6EAE4CE3E50E3DFEF825E2238C6126BE7AB9B1C6F141A68979605B_1497630299136_NDC+Sketch.jpg)


In our case, the viewport is 862x742 pixels, so:


     Pixel Coordinates: (NDC.x * width + width / 2, height / 2 - NDC.y * height)
                      = (0.4481 * 862 + 431, 371 - (-0.1688 * 742))
                      = (x: 624, y: 434)

🎉 


![screen coordinates](https://d2mxuefqeaa7sj.cloudfront.net/s_0240CF0C8C6EAE4CE3E50E3DFEF825E2238C6126BE7AB9B1C6F141A68979605B_1497564013914_Washington+Monument+Screen+Coordinates.png)