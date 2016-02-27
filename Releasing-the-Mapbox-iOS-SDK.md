## Version the packages

1. Choose a version number per [Semantic Versioning](http://semver.org/) and [our tagging rules](./Versions-and-tagging). Let's call it `ios-vX.Y.Z`. If this is a pre-release, go with `ios-vX.Y.Z-pre.P`, where `P` begins at `1` and increments for each pre-release. 
1. If necessary, update [the screenshot](https://github.com/mapbox/mapbox-gl-native/blob/master/platform/ios/screenshot.png).
1. Update the version [in the podspec](https://github.com/mapbox/mapbox-gl-native/blob/master/platform/ios/Mapbox-iOS-SDK.podspec#L4). 
  - Use `ios-vX.Y.Z` for stable releases to provide smaller, non-symbolicated downloads. 
  - Add the `-symbols` suffix when using intermediary dev releases to gather useful crash info (e.g. `ios-vX.Y.Z-pre.1-symbols`). This causes the larger, symbolicated install to be used in the integrating project. 
1. Update the `CHANGELOG.md` for the release.
1. Create a tag `ios-vX.Y.Z`.

## Build and deploy the packages

```bash
export TRAVIS_REPO_SLUG=mapbox-gl-native
mbx auth …
# Append -pre.P for prerelease P:
export PUBLISH_VERSION=X.Y.Z
make clean && make distclean
make ipackage
./platform/ios/scripts/publish.sh "${PUBLISH_VERSION}" symbols
make ipackage-strip
./platform/ios/scripts/publish.sh "${PUBLISH_VERSION}"
make iframework
./platform/ios/scripts/publish.sh "${PUBLISH_VERSION}" symbols-dynamic
make iframework SYMBOLS=NO
./platform/ios/scripts/publish.sh "${PUBLISH_VERSION}" dynamic
```

## Release the packages

```bash
# Download the packages from s3 to make sure they work.
git push
git push --tags
open https://github.com/mapbox/mapbox-gl-native/releases/new
# Add release notes and attach the packages to the release.

# For stable releases:
pod trunk push
# Repackage the static framework bundle for Fabric distribution.
```

Now update the documentation.