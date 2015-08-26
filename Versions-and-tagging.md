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

## Core tags

**In addition** to any per-platform tags, we should also tag core itself (again, following Semantic Versioning) on the same commit. 

Example: `v0.5.5 = ios-v2.1.0-pre.1 = 85124b8`