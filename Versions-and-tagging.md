In all cases, we follow [Semantic Versioning](http://semver.org) to indicate compatibility, breaking changes, and other intentions. 

## Prefix

All version numbers should have a `v` prefix. 

Example: `v0.5.4`

## Per-platform tags

For iOS, Android, Node.js, and other per-platform bindings that are integrated into this repository, we use prefixes on tags to indicate platform-specific versions as required by tagging needs or marketing. 

Examples: 

- `ios-v2.0.0-pre.1`
- `android-v0.1.3`
- `node-v2.0.0`

For the Mapbox iOS and OS X SDKs, the major version is incremented whenever a public API is changed in a way that breaks backwards compatibility, particularly when a symbol is renamed or removed, but also potentially when the semantics of a symbol change significantly. Changes to the minimum deployment target are made with great care but do not necessarily warrant a major version bump.

## Core tags

In the past, we used to also tag core itself (again, following Semantic Versioning) on the same commit. We no longer do so because all the SDKs are in the same repository, making it much easier to make cross-platform changes without worrying about versioning.

Example: `v0.5.5 = ios-v2.1.0-pre.1 = 85124b8`