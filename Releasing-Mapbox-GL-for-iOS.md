## GitHub

1. Choose a version number per [Semantic Versioning](http://semver.org/). Let's call it `X.Y.Z`.
1. If necessary, update [the screenshot](https://github.com/mapbox/mapbox-gl-native/blob/master/ios/screenshot.png).
1. Update the version [in the podspec](https://github.com/mapbox/mapbox-gl-native/blob/master/MapboxGL.podspec#L4). 
1. Push those two changes.
1. Create a tag `ios-vX.Y.Z` and push the tag. 
1. Create and push a deploy commit. 

```bash
$ git commit --allow-empty -m '[publish ios-vX.Y.Z]'
```

## CocoaPods

Your app's `Podfile` should be as follows (plus any other pods used): 

```ruby
pod 'MapboxGL', :git => 'https://github.com/mapbox/mapbox-gl-native'

use_frameworks!
```

Then, to update to the latest published version: 

```bash
$ pod update
```

There is no way to pin versions right now; that will come when we release officially and on the CocoaPods master repository. 