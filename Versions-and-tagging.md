In all cases, we follow [Semantic Versioning](http://semver.org) to indicate compatibility, breaking changes, and other intentions. 

All version numbers should have a `v` prefix. For iOS, Android, Node.js, and other per-platform bindings that are integrated into this repository, we use an additional prefix on each tag to indicate a platform-specific version as required by tagging needs or marketing.

Examples:

- `ios-v3.4.0-beta.1`
- `android-v0.1.3`
- `node-v2.0.0`

## Mapbox Android SDK

The Android SDK currently offers three products (from less stable to more stable):

1. `SNAPSHOT` releases: built nightly automatically. E.g., `v4.2.0-SNAPSHOT`.
2. `beta` releases: Our main pre-release vehicle. E.g., `v4.2.0-beta.1`.
3. Final release: The recommended product for our developers in production. E.g., `v4.2.0`

Releases, betas, and snapshots are all built in release mode, by the same build environment under the same [settings](https://github.com/mapbox/mapbox-gl-native/blob/master/platform/android/bitrise.yml). The only difference between them is its version naming (and therefore, level of stability and available features).

They all are distributed either via Maven or Sonatype, like any other Java/Android package. You can get instructions on how to install them in your app in our [documentation page](https://www.mapbox.com/android-sdk/).

### Final releases

Final releases are the recommended product to ship with your apps in the app store. For the Mapbox Android SDK, the major version is incremented whenever a public, documented API is changed in a way that breaks backwards compatibility. E.g `v1.2.3` would be followed by `v2.0.0`.

### Pre-releases

In preparation for a release we will ship a number of betas with increased stability (at this moment, we don't use `alpha` or `rc`). All betas are publicly ticketed in this repo containing information about their features, known issues, and level of stability ([example](https://github.com/mapbox/mapbox-gl-native/issues/6418)). Betas are a great mechanism to get your apps ready before a final release, and to provide timely feedback to the Mapbox team to help us shape the final APIs and feature-set.

### Snapshots

Finally, we publish nightly snapshots. These are not intended for production use, instead they're useful to troubleshoot issues as they're easier to use than building this repo from scratch. Nightly snapshots are built from the `master` branch and include all the PRs changes landed the previous day.


## Mapbox iOS SDK

For the Mapbox iOS SDK, the major version is incremented whenever a public, documented API is changed in a way that breaks backwards compatibility, particularly when a symbol is renamed or removed, but also potentially when the semantics of a symbol change significantly. Changes to the installation process or minimum deployment target also warrant a major version bump.

As much as possible, symbols are marked deprecated long before being marked unavailable, long before being removed outright. Undocumented APIs – even publicly visible ones – are subject to change without notice or major version bump. Undocumented behavior in documented APIs is handled on a case-by-case basis. Changes that only affect Swift bridging are also handled on a case-by-case basis.

We tag alpha releases to provide development snapshots as we work towards a beta. Alphas are unstable and should not be used for production development. We issue a beta release once the major features have landed and we feel confident that a developer can begin working with these features in a shippable application, with an eye towards using the final SDK release in the final application. New APIs may be fluid and unstable until the release candidates. We issue a release candidate once we’ve fixed all the bugs we intend to fix for the final release (allowing for the possibility that additional bugs may be found in the RC).

## Mapbox macOS SDK

The Mapbox macOS SDK has yet to reach version 1.0. Therefore, backwards-incompatible changes may be introduced in every release.

## Mapbox Qt SDK

The Mapbox Qt SDK has yet to be tagged for a release or prerelease.

## Core tags

In the past, we used to also tag core itself (again, following Semantic Versioning) on the same commit. We no longer do so because all the SDKs are in the same repository, making it much easier to make cross-platform changes without worrying about versioning.

Example: `v0.5.5 = ios-v2.1.0-pre.1 = 85124b8`