Follow [these guidelines](Release-notes-style-guide) for drafting release notes as part of the running changelog.

* Replace `0.9.8-alpha.1` with the macOS SDK release version.
* Replace `9.8.7-alpha.1` with the corresponding iOS SDK release version.
* Replace `0.9.7` with the previous macOS SDK release version.
* Insert the most recent section of [this changelog](https://github.com/mapbox/mapbox-gl-native/blob/master/platform/macos/CHANGELOG.md) into the middle portion of the template, adding blurbs for changes since the last prerelease.

```markdown
This version of the Mapbox Maps SDK for macOS corresponds to version 9.8.7-alpha.1 of the Mapbox Maps SDK for iOS. [Changes](https://github.com/mapbox/mapbox-gl-native/compare/macos-v0.9.7...macos-v0.9.8-alpha.1) since [macos-v0.9.7](https://github.com/mapbox/mapbox-gl-native/releases/tag/macos-v0.9.7):

## Packaging

* 
* 
* 

## Styles and data

* 
* 
* 

## User location

* 
* 
* 

## Annotations

* 
* 
* 

## User interaction

* 
* 
* 

## Offline maps

* 
* 
* 

## Networking

* 
* 
* 

## Other changes

* 
* 
* 

To install this prerelease via CocoaPods, point your Podfile to either of these URLs:

* https://raw.githubusercontent.com/mapbox/mapbox-gl-native/macos-v0.9.8-alpha.1/platform/macos/Mapbox-macOS-SDK.podspec
* https://raw.githubusercontent.com/mapbox/mapbox-gl-native/macos-v0.9.8-alpha.1/platform/macos/Mapbox-macOS-SDK-symbols.podspec

Documentation is [available online](https://mapbox.github.io/mapbox-gl-native/macos/0.9.8-alpha.1/) or as part of the download.
```