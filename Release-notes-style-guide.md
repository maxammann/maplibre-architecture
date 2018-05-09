Release notes are the most visible form of documentation for the Mapbox Map SDK for iOS and macOS, so we take care to write it well and comprehensively. We maintain changelogs on a running basis: if a change is notable enough to appear in the release notes, then the pull request making the change should also update the changelog, following the guidelines below. That way, publishing an SDK version is a simple affair:

* For **macOS releases**, insert the most recent section of [this changelog](https://github.com/mapbox/mapbox-gl-native/blob/master/platform/macos/CHANGELOG.md) into [this template](Release-notes-template-for-macOS). (The template looks similar for iOS.)
* For **macOS prereleases**, insert the most recent section of [this changelog](https://github.com/mapbox/mapbox-gl-native/blob/master/platform/macos/CHANGELOG.md) into [this template](Prerelease-notes-template-for-macOS). (The template looks similar for iOS.)

## Foreword

The forward provides helpful links to related tags. When necessary, it also highlights important compatibility notes, such as the corresponding iOS map SDK version or upcoming compatibility changes.

## Blurbs

Include only changes that impact developers or end users:

* For final releases: list notable changes since the previous release. This list should be cumulative across all the intervening prereleases. To avoid confusion, omit fixes for regressions that were introduced since the previous release.
* For prereleases: list notable changes since the previous release or prerelease. Omit fixes for regressions that were introduced since the previous prerelease, but include fixes for regressions that were present in the preview prerelease. Those fixes may not be listed in the changelog, so you’ll have to make up something on the spot.

### Categories

For readability, sort the list of changes into several categories:

* If any category has fewer than three items, combine it with another category or combine it with “Other changes”.
* If there are fewer than three categories, use a single, flat list of changes.
* Within each category, sort blurbs in order of descending developer or end user impact. List backwards-incompatible changes towards the top.

Each blurb describes a single change that is notable from the developer’s or end user’s perspective:

* Focus on what changed, not how we changed it.
* Use complete sentences, but omit the subject “we” to avoid repetitiveness. Cast sentences in the past or past progressive tense, since this is a historical record.
* Refer to symbols in backticks so that jazzy can automatically link them to the relevant documentation.
* Link to the pull request that implemented the change. For cherry-picked changes, link to the original PR.

### Example blurbs

* Added the `-[MGLImageSource enhance]` method for enhancing an image source. You can also set the source’s `enhance` property to `true` in style JSON. (#123)
* Fixed an issue that sometimes turned the user location view into a teapot. (#456)
* Latitudes no longer come after longitudes when formatting `CLLocationCoordinate2D` instances. To preserve the original behavior, pay a visit to Null Island. (#789)

## Afterword

The afterword points the developer to the published documentation. Ensure that a jazzy docset is located at the linked URL.

For prereleases, note the relevant CocoaPods podspec URLs.