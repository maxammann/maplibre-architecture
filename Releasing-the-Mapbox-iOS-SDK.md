(Consider [releasing the Mapbox macOS SDK](https://github.com/mapbox/mapbox-gl-native/wiki/Releasing-the-Mapbox-macOS-SDK) at the same time.)

## Version the packages

1. Choose a version number per [Semantic Versioning](http://semver.org/) and [our tagging rules](./Versions-and-tagging). Let's call it `ios-vX.Y.Z`. If this is a pre-release, go with `ios-vX.Y.Z-pre.P`, where `P` begins at `1` and increments for each pre-release. 
1. If necessary, update [the screenshot](https://github.com/mapbox/mapbox-gl-native/raw/ios-v3.6.0/platform/ios/docs/img/screenshot.png).
1. Update the version [in the podspec](https://github.com/mapbox/mapbox-gl-native/blob/ios-v3.6.0/platform/ios/Mapbox-iOS-SDK.podspec#L3), [-symbols podspec](https://github.com/mapbox/mapbox-gl-native/blob/ios-v3.6.0/platform/ios/Mapbox-iOS-SDK-symbols.podspec#L3), and [-nightly-dynamic podspec](https://github.com/mapbox/mapbox-gl-native/blob/ios-v3.6.0/platform/ios/Mapbox-iOS-SDK-nightly-dynamic.podspec#L3).
1. Update the `CHANGELOG.md` for the release.
   - Add today’s date to the header for the release.
   - #protip: you can use the compare (`ios-v#.#.#-previous-beta.#...release-N|master`) feature in github to more easily find intra-release changes (i.e. https://github.com/mapbox/mapbox-gl-native/compare/ios-v3.3.0-alpha.2...ios-v3.3.0-alpha.3).
1. Run `tx pull -a` to [add or update translations](https://github.com/mapbox/mapbox-gl-native/blob/master/platform/ios/DEVELOPING.md#adding-a-localization).
1. Create a pull request with these changes and have it approved/merged.
1. Create a tag `ios-vX.Y.Z`.
1. `git push origin ios-vX.Y.Z`

## Build and release

The release build and deployment process starts [on Bitrise](https://www.bitrise.io/app/7514e4cf3da2cc57) once you push the tag. This will automatically:

- Build, package, and upload the different release flavors to s3 and GitHub.
- Create a draft release [on GitHub](https://github.com/mapbox/mapbox-gl-native/releases).

Once the ~50 minute deployment process is finished, you should:

- Copy the release notes to the draft GitHub release.
- Check that the attached packages are valid.
- Push the publish button. 💚

### The old manual way

In times of dire need, you can still deploy manually by following the instructions in [this old revision](https://github.com/mapbox/mapbox-gl-native/wiki/Releasing-the-Mapbox-iOS-SDK/8b2f745e62ebde5b8663e5016fe7b50072eaad77), which relies on [this script](https://github.com/mapbox/mapbox-gl-native/blob/master/platform/ios/scripts/deploy-packages.sh).

## Stable releases

### cocoapods

##### Note: You should first [register a CocoaPods id](https://guides.cocoapods.org/making/getting-setup-with-trunk.html#getting-started) and [be added as a CocoaPods collaborator](https://guides.cocoapods.org/making/getting-setup-with-trunk.html#adding-other-people-as-contributors) for the [Mapbox-iOS-SDK pod](https://cocoapods.org/?q=Mapbox-iOS-SDK)

- Run `pod trunk push platform/ios/Mapbox-iOS-SDK.podspec`.

### Documentation

- [Update the documentation.](https://github.com/mapbox/gl-internal/wiki/Updating-documentation-on-release)

## Pre-releases

### Documentation

Publish API documentation in the [mapbox/ios-sdk](https://github.com/mapbox/ios-sdk) repo. After generating the docs, only commit the new `api/X.X.X/` folder — this makes them publicly available, but leaves the stable version as the default.