Here's how you can prototype your mobile app in Framer.js with real Mapbox maps. This way your integrated team can move fast on design decisions and use real map styles from day 1.

Mapbox is the best tool for integrated teams of mobile developers, UX designers, and graphic designers. We want to support apps that are custom to the core and tuned exactly for your app. If you already use Framer.js for this purpose, this tutorial should fit right into your process.

The intent of this tutorial is to integrate Mapbox GL JS into your Framer prototypes. For your native apps, you'll use the [Mapbox iOS SDK](http://mapbox.com/ios-sdk) and [Mapbox Android SDK](https://www.mapbox.com/android-sdk/) for that truly native performance, but since Framer is based on web technologies, our JavaScript code will work perfectly.

To get started, you'll need:

* [Framer.js](http://framerjs.com/)
* [nodejs & npm](https://nodejs.org/en/)

If you haven't used a terminal before, the first step will be the hardest, but it'll be all fun after that.

## Installing

You can either have an existing Framer project, or create a new, blank one. Framer projects exist on your computers as folders named something like `YourProject.framer`, and contain CoffeeScript source code and a directory called `modules`. We're going to install Mapbox GL JS into your project as a module.

Open a [terminal](http://blog.teamtreehouse.com/introduction-to-the-mac-os-x-command-line) and navigate to your project folder. If your project is in your Documents folder instead of in your home folder, you'd type `~/Documents/my-framer-project`

```sh
$ cd ~/my-framer-project
```

Next we'll install the latest version of Mapbox GL JS, using npm:

```sh
$ npm install mapbox-gl
```

Great: once this is done we have the Mapbox GL JS source code downloaded. To access it from your project, though, we'll need a file that connects it. Create a new file in the `modules` directory inside of your project and call it `npm.coffee`. Give it these contents:

```coffeescript
exports.mapboxgl = require "mapbox-gl"
```

Now your project will be able to require Mapbox GL JS. This is something you'll do at the top of your Framer project, a single line that tells the project to pull in the `mapboxgl` object, making mapping available.

```coffeescript
{ mapboxgl } = require "npm"
```

Once you have `mapboxgl` imported, you can start adding maps.

Two notes before we pull together that source code:

* Maps are added in HTML elements, and by default Framer makes HTML non-interactive. This is why we add the line `mapboxLayer.ignoreEvents = false` - this turns events back on, so you can pan and zoom the map. You can omit this line if you want maps to be non-interactive.
* Framer scales prototypes at a number of ratios - you might look at your prototype at 75% scale to view a high-resolution mockup on a small laptop screen. Mapbox GL JS will act a little strangely at scales that aren't 100% - panning the map might seem a slower. You can count on the map behaving correctly at 100% scale.

```coffeescript
{ mapboxgl } = require "npm"


# Here we're creating a new HTML layer
# for the map to live inside of, and scaling
# it to fit the entire window
mapHeight = Screen.height
mapWidth = Screen.width

mapboxLayer = new Layer
mapboxLayer.ignoreEvents = false
mapboxLayer.width = mapWidth
mapboxLayer.height = mapHeight
mapboxLayer.html = "<div id='map'></div>"
mapElement = mapboxLayer.querySelector("#map")
mapElement.style.height = mapHeight + 'px'

# Set your Mapbox access token here!
mapboxgl.accessToken = '{your_access_token}'

map = new mapboxgl.Map({
    container: mapElement
    zoom: 12.5
    center: [-77.01866, 38.888]
    # here we're using a default style:
    # you can use any of the defaults or a
    # custom style you design in Mapbox Studio
    style: 'mapbox://styles/mapbox/streets-v8'
    hash: true
})
```

That's it! The `map` object you have access to in Framer will have the same methods and properties as [the Mapbox GL JS API](http://mapbox.com/mapbox-gl-js).

Have fun prototyping!