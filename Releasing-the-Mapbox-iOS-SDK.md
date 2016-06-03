## Version the packages

1. Choose a version number per [Semantic Versioning](http://semver.org/) and [our tagging rules](./Versions-and-tagging). Let's call it `ios-vX.Y.Z`. If this is a pre-release, go with `ios-vX.Y.Z-pre.P`, where `P` begins at `1` and increments for each pre-release. 
1. If necessary, update [the screenshot](https://github.com/mapbox/mapbox-gl-native/blob/master/platform/ios/screenshot.png).
1. Update the version [in the podspec](https://github.com/mapbox/mapbox-gl-native/blob/master/platform/ios/Mapbox-iOS-SDK.podspec#L4) and [-symbols podspec](https://github.com/mapbox/mapbox-gl-native/blob/master/platform/ios/Mapbox-iOS-SDK-symbols.podspec#L4).
  - Use `X.Y.Z{-alpha|beta.P}` to provide smaller, non-symbolicated downloads. 
  - The `-symbols` suffix is used in [-symbols podspec](https://github.com/mapbox/mapbox-gl-native/blob/master/platform/ios/Mapbox-iOS-SDK-symbols.podspec#L4) for intermediary dev releases to gather useful crash info (e.g. `X.Y.Z{-alpha|beta.P}-symbols`). This causes the larger, symbolicated install to be used in the integrating project. 
1. Update the `CHANGELOG.md` for the release. #protip: you can use the compare feature in github to more easily find intra-release changes (i.e. https://github.com/mapbox/mapbox-gl-native/compare/ios-v3.3.0-alpha.2...ios-v3.3.0-alpha.3)
1. Create a tag `ios-vX.Y.Z`.
1. `git push`
1. `git push --tags`

## Build and release

You can follow the manual instructions in [this gist](https://gist.github.com/boundsj/5fadf57e5114de4d45c3c4af40f9836e). However we expect to deprecate that approach in the future in favor of a more automated approach on a CI server. In the interim, [a script](https://github.com/mapbox/mapbox-gl-native/blob/master/platform/ios/scripts/deploy-packages.sh) automates most of the work so you can follow these simple steps:

- Run `mbx auth ...` (if you are new, ask a team member for help with this)
- Add `GITHUB_TOKEN` environment variable to be able to create a github release from the command line with [github-release](https://github.com/aktau/github-release) (again ask a team member for help if this does not make sense)
- Run `./platform/ios/scripts/deploy-packages.sh {major}.{minor}.{patch}{-alphatag.N} ~/path/to/download -g` (e.g. `./platform/ios/scripts/deploy-packages.sh 42.42.42-alpha.1 ~/Downloads/tmp -g`). This will:
 - Build all the packages (static and dynamic framework files and friends).
 - Upload to s3 (if you've run `mbx auth` above).
 - Test that downloads from s3 work.
 - Make a new Github release draft and upload all of the compressed release files to the Github release (if you've set your `GITHUB_TOKEN` as noted above).
- Go to https://github.com/mapbox/mapbox-gl-native/releases to find the draft, confirm that it is valid, and add notes from the changelog.
- When you are satisfied with the release draft, click the button to publish it.

## For stable releases:

###### cocoapods

- Run `pod trunk push` (if you are new, ask a team member to add you to the pods collaborator list)

###### Fabric

- Repackage the static framework bundle for Fabric distribution, test, and release:
  - open fabric.io/kits/manage
  - make a new release
  - add the compressed result of `make ifabric` (Mapbox.framework.zip)
  - make any other required adjustment to the release meta data then click "Submit for Review"
  - Test:
    - Make a new app (or update existing) app Mapbox framework with the fabric osx app making sure to get the new version you just submitted for review
    - As noted in the Fabric osx app, perform ⌘R to run your app -- verify that the Fabric app is satisfied with the result
    - Go back to the fabric.io/kits "new release" page and click the button to publish

###### Documentation

- [update the documentation](https://github.com/mapbox/gl-internal/wiki/Updating-documentation-on-release).