## GitHub

1. Choose a version number per [Semantic Versioning](http://semver.org/) and [our tagging rules](../Versions-and-tagging). Let's call it `ios-vX.Y.Z`. If this is a pre-release, go with `ios-vX.Y.Z-pre.P`, where `P` begins at `1` and increments for each pre-release. 
1. If necessary, update [the screenshot](https://github.com/mapbox/mapbox-gl-native/blob/master/ios/screenshot.png).
1. Update the version [in the podspec](https://github.com/mapbox/mapbox-gl-native/blob/master/ios/MapboxGL.podspec#L4). 
  - Use `ios-vX.Y.Z` for stable releases to provide smaller, non-symbolicated downloads. 
  - Add the `-symbols` suffix when using intermediary dev releases to gather useful crash info (e.g. `ios-vX.Y.Z-pre.1-symbols`). This causes the larger, symbolicated install to be used in the integrating project. 
1. Update the version in `+[MGLAccountManager kitDisplayVersion]` for Fabric. 
1. Push those changes.
1. Create a tag `ios-vX.Y.Z` and push the tag. 

### Automatic deploy steps currently not working

1. Create and push a deploy commit with `git commit --allow-empty -m '[publish ios-vX.Y.Z]'`. 
1. When the publisher has created and uploaded zip files (stripped as well as with debug symbols; URLs obtained from the Travis publisher output log) to S3, publish them to [releases](https://github.com/mapbox/mapbox-gl-native/releases/new) as well. We'll automate this soon. 

### Other

1. If this is a public, stable release, push to CocoaPods with `pod trunk push`.
1. Update docs at `mapbox.com/ios-sdk`. 