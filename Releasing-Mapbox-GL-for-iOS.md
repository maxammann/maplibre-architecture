## GitHub

1. Choose a version number per [Semantic Versioning](http://semver.org/). Let's call it `X.Y.Z`.
1. If necessary, update [the screenshot](https://github.com/mapbox/mapbox-gl-native/blob/master/ios/screenshot.png).
1. Update the version [in the podspec](https://github.com/mapbox/mapbox-gl-native/blob/master/ios/MapboxGL.podspec#L4). 
  - Use `X.Y.Z` for stable releases to provide smaller, non-symbolicated downloads. 
  - Use `X.Y.Z-symbols` for intermediary dev releases to gather useful crash info. 
1. Push those two changes.
1. Create a tag `vX.Y.Z` and push the tag. 
1. Create and push a deploy commit with `git commit --allow-empty -m '[publish ios-vX.Y.Z]'`. Note the `ios-` prefix, which tells the publisher to build for iOS.  
1. When the publisher has created and uploaded zip files (stripped as well as with debug symbols) to S3, publish them to [releases](https://github.com/mapbox/mapbox-gl-native/releases/new) as well. We'll automate this soon. 

Now you're ready to [install Mapbox GL](Installing Mapbox GL for iOS).