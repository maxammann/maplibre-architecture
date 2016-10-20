In all cases, we follow [Semantic Versioning](http://semver.org) to indicate compatibility, breaking changes, and other intentions. 

## Prefix

All version numbers should have a `v` prefix. 

Example: `v0.5.4`

## Per-platform tags

For iOS, Android, Node.js, and other per-platform bindings that are integrated into this repository, we use prefixes on tags to indicate platform-specific versions as required by tagging needs or marketing. 

Examples: 

- `ios-v3.4.0-beta.1`
- `android-v0.1.3`
- `node-v2.0.0`

### Mapbox Android SDK

For the Mapbox Android SDK, the major version is incremented whenever a public, documented API is changed in a way that breaks backwards compatibility.

Nightly snapshots and beta releases are not intended for production use.

### Mapbox iOS SDK

For the Mapbox iOS SDK, the major version is incremented whenever a public, documented API is changed in a way that breaks backwards compatibility, particularly when a symbol is renamed or removed, but also potentially when the semantics of a symbol change significantly.

Changes to the installation process or minimum deployment target are made with great care but do not necessarily warrant a major version bump. As much as possible, symbols are marked deprecated long before being marked unavailable, long before being removed outright. Undocumented APIs – even publicly visible ones – are subject to change without notice or major version bump. Undocumented behavior in documented APIs is handled on a case-by-case basis. Changes that only affect Swift bridging are also handled on a case-by-case basis.

We tag alpha releases to provide development snapshots as we work towards a beta. Alphas are unstable and should not be used for production development. We issue a beta release once the major features have landed and we feel confident that a developer can begin working with these features in a shippable application, with an eye towards using the final SDK release in the final application. New APIs may be fluid and unstable until the release candidates. We issue a release candidate once we’ve fixed all the bugs we intend to fix for the final release (allowing for the possibility that additional bugs may be found in the RC).

### Mapbox macOS SDK

The Mapbox macOS SDK has yet to reach version 1.0. Therefore, backwards-incompatible changes may be introduced in every release.

## Core tags

In the past, we used to also tag core itself (again, following Semantic Versioning) on the same commit. We no longer do so because all the SDKs are in the same repository, making it much easier to make cross-platform changes without worrying about versioning.

Example: `v0.5.5 = ios-v2.1.0-pre.1 = 85124b8`