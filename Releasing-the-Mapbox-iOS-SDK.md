(Consider [releasing the Mapbox macOS SDK](https://github.com/mapbox/mapbox-gl-native/wiki/Releasing-the-Mapbox-macOS-SDK) at the same time.)

## Version the packages

1. Choose a version number per [Semantic Versioning](http://semver.org/) and [our tagging rules](./Versions-and-tagging). Let's call it `ios-vX.Y.Z`. If this is a pre-release, go with `ios-vX.Y.Z-pre.P`, where `P` begins at `1` and increments for each pre-release. 
1. If necessary, update [the screenshot](https://github.com/mapbox/mapbox-gl-native/blob/master/platform/ios/screenshot.png).
1. Update the version [in the podspec](https://github.com/mapbox/mapbox-gl-native/blob/master/platform/ios/Mapbox-iOS-SDK.podspec#L4) and [-symbols podspec](https://github.com/mapbox/mapbox-gl-native/blob/master/platform/ios/Mapbox-iOS-SDK-symbols.podspec#L4).
  - Use `X.Y.Z{-alpha|beta.P}` to provide smaller, non-symbolicated downloads. 
  - The `-symbols` suffix is used in [-symbols podspec](https://github.com/mapbox/mapbox-gl-native/blob/master/platform/ios/Mapbox-iOS-SDK-symbols.podspec#L4) for intermediary dev releases to gather useful crash info (e.g. `X.Y.Z{-alpha|beta.P}-symbols`). This causes the larger, symbolicated install to be used in the integrating project. 
1. Update the `CHANGELOG.md` for the release.
  - #protip: you can use the compare (`ios-v#.#.#-previous-beta.#...release-N|master`) feature in github to more easily find intra-release changes (i.e. https://github.com/mapbox/mapbox-gl-native/compare/ios-v3.3.0-alpha.2...ios-v3.3.0-alpha.3).
1. Run `tx pull -a` to [add or update translations](https://github.com/mapbox/mapbox-gl-native/blob/master/platform/ios/DEVELOPING.md#adding-a-localization).
1. Create a pull request with these changes and have it approved/merged.
1. Create a tag `ios-vX.Y.Z`.
1. `git push origin ios-vX.Y.Z`

## Build and release

You can follow the manual instructions in [this gist](https://gist.github.com/boundsj/5fadf57e5114de4d45c3c4af40f9836e). However, we expect to deprecate that approach in the future in favor of a more automated approach on a CI server. In the interim, [a script](https://github.com/mapbox/mapbox-gl-native/blob/master/platform/ios/scripts/deploy-packages.sh) automates most of the work so you can follow these simple steps:

- Run `mbx auth <your-2fa-code>` _(If you do not already have AWS credentials, ask a team member for help in setting this up.)_
- _[First time only]_ To create a GitHub release from the command line, you will need to:
   - [Create a new GitHub access token](https://help.github.com/articles/creating-an-access-token-for-command-line-use/) and add it as the `GITHUB_TOKEN` environment variable — e.g., `export GITHUB_TOKEN='8BADF00DDEADBEEFC00010FF'` in your `~/.bash_profile`.
- Run `make ideploy`. This will:
 - Build all the packages (static and dynamic framework files and friends).
 - Upload to s3 (if you've run `mbx auth` above).
 - Test that downloads from s3 work.
 - Make a new Github release draft and upload all of the compressed release files to the Github release (if you've set your `GITHUB_TOKEN` as noted above).
- Go to https://github.com/mapbox/mapbox-gl-native/releases to find the draft, confirm that it is valid, and add notes from the changelog.
- When you are satisfied with the release draft, click the button to publish it.

## Stable releases

### cocoapods

##### Note: You should first [register a CocoaPods id](https://guides.cocoapods.org/making/getting-setup-with-trunk.html#getting-started) and [be added as a CocoaPods collaborator](https://guides.cocoapods.org/making/getting-setup-with-trunk.html#adding-other-people-as-contributors) for the [Mapbox-iOS-SDK pod](https://cocoapods.org/?q=Mapbox-iOS-SDK)

- Run `pod trunk push platform/ios/Mapbox-iOS-SDK.podspec`.

### Documentation

- [Update the documentation.](https://github.com/mapbox/gl-internal/wiki/Updating-documentation-on-release)

## Pre-releases

### Documentation

Publish API documentation in the [mapbox/ios-sdk](https://github.com/mapbox/ios-sdk) repo. After generating the docs, only commit the new `api/X.X.X/` folder — this makes them publicly available, but leaves the stable version as the default.