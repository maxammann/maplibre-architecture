Congratulations, you’re the proud owner of a new Mapbox macOS SDK release! The process below takes 20–30 minutes.

## When to release

To the extent possible, we release the Mapbox macOS SDK in tandem with the [Mapbox iOS SDK](https://github.com/mapbox/mapbox-gl-native/wiki/Releasing-the-Mapbox-iOS-SDK), so that developers who work on both platforms can count on feature parity and the same bug fixes between the two SDKs. However, if a critical bug fix affects only macOS, we may issue an out-of-band release of the macOS SDK; conversely, we may delay the macOS SDK release if a critical issue arises in the iOS SDK.

## Prepare for release

_Takes under 5 minutes_

1. Skim the [issues tagged <kbd>macOS</kbd>](https://github.com/mapbox/mapbox-gl-native/issues?q=is%3Aopen+is%3Aissue+label%3AmacOS) or [issues mentioning “macOS”](https://github.com/mapbox/mapbox-gl-native/issues?utf8=✓&q=is%3Aopen%20is%3Aissue%20macOS%20) in case there are any showstoppers.
1. Run `tx pull -a` to [add or update translations](https://github.com/mapbox/mapbox-gl-native/blob/master/platform/macos/DEVELOPING.md#adding-a-localization).
1. Update the first section of [the changelog](https://github.com/mapbox/mapbox-gl-native/blob/master/platform/macos/CHANGELOG.md) to reflect any notable changes to the SDK, shared iOS/macOS code, or mbgl since the previous release. [Compare](https://github.com/mapbox/mapbox-gl-native/compare/) the previous release tag with master or the release branch to find notable changes. Look in particular for commit messages prefixed with `[macos]`, `[ios, macos]`, or `[core]`.
1. If [the screenshot](https://github.com/mapbox/mapbox-gl-native/blob/master/platform/macos/docs/img/screenshot.jpg) hasn’t been updated in awhile, compose a new version that shows off the new release’s features. Try to show styles, cities, or features [that haven’t been depicted](https://github.com/mapbox/mapbox-gl-native/issues/7877) in the macOS SDK screenshot before.
1. Commit these changes and open a PR to get them reviewed and merged.

## Tag the release

_Takes under 5 minutes_

1. Decide on a semver-compliant version number according to [these guidelines](https://github.com/mapbox/mapbox-gl-native/wiki/Versions-and-tagging). The version number should be of the form 0.9.8-alpha.1, 0.9.8-beta.1, 0.9.8-rc.1 (for a release candidate), or 0.9.8 (for a final release).
1. Update the `version` variable in [Mapbox-macOS-SDK.podspec](https://github.com/mapbox/mapbox-gl-native/blob/master/platform/macos/Mapbox-macOS-SDK.podspec) and [Mapbox-macOS-SDK-symbols.podspec](https://github.com/mapbox/mapbox-gl-native/blob/master/platform/macos/Mapbox-macOS-SDK-symbols.podspec). Commit this change with a commit message like `macos-v0.9.8` and open a PR to get it reviewed and merged.
1. Tag the merged podspec changes as `macos-v0.9.8`, where _0.9.8_ is the semver-compliant version you chose in step 1. Push the tag.

## Publish the release

_Takes 10–15 minutes_

1. Run `xcodebuild -version` or `xcode-select -p` to make sure you’re building with the right version of Xcode.
1. If this is your first time releasing the Mapbox macOS SDK:
   1. Install `wget` and [`github-release`](https://github.com/aktau/github-release/) from Homebrew and [`caffeinate`](https://rubygems.org/gems/caffeinate) from RubyGems.
   1. Add `export GITHUB_TOKEN='8BADF00DDEADBEEFC00010FF'` to your .bash_profile, where _8BADF00DDEADBEEFC00010FF_ is a [new GitHub personal access token](https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/).
   1. Optionally, to code-sign Mapbox GL.app, you’ll need to install a provisioning profile for your Mac Developer Program membership.
1. Make sure your Mac is plugged in, then run `make xdeploy`. A script will build the SDK, package it up, and finally upload the package to a new GitHub release, all the while keeping your computer from falling asleep.
1. While you wait, [draft a new release](https://github.com/mapbox/mapbox-gl-native/releases/new/). Add release notes based on the release’s section in the changelog. Use one of these templates:
   * [Release notes template](https://github.com/mapbox/mapbox-gl-native/wiki/Release-notes-template-for-macOS)
   * [Prerelease notes template](https://github.com/mapbox/mapbox-gl-native/wiki/Prerelease-notes-template-for-macOS)
1. Once the script runs to completion, it should have drafted a new [GitHub release](https://github.com/mapbox/mapbox-gl-native/releases/) with binary packages attached. Copy the release notes you drafted above into the new release. Title the release `macos-v0.9.8` (where _0.9.8_ is the new version). Save the draft.
1. Optionally, build Mapbox GL.app:
   1. Run `make xproj` and switch to the macosapp scheme.
   1. Go to Product ‣ Archive.
   1. After macosapp finishes archiving, go to the Archives tab of the Organizer window. Select the macosapp archive, then click Export in the right sidebar.
   1. In the sheet that appears, select “Export a Developer ID–signed Application”.
   1. Zip up the exported .app and attach it to the release draft.
1. Check “This is a pre-release” if applicable, then click “Publish release”. Delete the other draft you made.

## Update the documentation

_Takes 5–10 minutes_

1. Update the [Mapbox macOS SDK documentation site](https://mapbox.github.io/mapbox-gl-native/macos/) (which is also bundled with the SDK):
   1. Clone mapbox-gl-native to a mapbox-gl-native-pages folder alongside your main mapbox-gl-native clone, and check out the `gh-pages` branch.
   1. In your main mapbox-gl-native clone, check out the release branch and run `make xdocument STANDALONE=1 OUTPUT=../mapbox-gl-native-pages/macos/0.9.8`, where _0.9.8_ is the new SDK version.
   1. In mapbox-gl-native-pages, edit [macos/index.html](https://github.com/mapbox/mapbox-gl-native/blob/gh-pages/macos/index.html) and macos/docsets/Mapbox.xml to refer to the new SDK version.
   1. In mapbox-gl-native-pages, edit [macos/Mapbox-macOS-SDK.json](https://github.com/mapbox/mapbox-gl-native/blob/gh-pages/macos/Mapbox-macOS-SDK.json) and [macos/Mapbox-macOS-SDK-symbols.json](https://github.com/mapbox/mapbox-gl-native/blob/gh-pages/macos/Mapbox-macOS-SDK-symbols.json).
   1. Commit and push your changes to the `gh-pages` branch.
1. If any new style properties or features are supported in the new release, update the “SDK support” tables in the [style specification documentation](https://www.mapbox.com/mapbox-gl-js/style-spec/):
   1. If this is your first time updating the style specification documentation, install `yarn` from Homebrew.
   1. Update the `sdk-support` objects in [v8.json](https://github.com/mapbox/mapbox-gl-js/blob/master/src/style-spec/reference/v8.json). The `macos` key in an `sdk-support` object for a particular property indicates the minimum macOS SDK version that supports that property. If the `sdk-support` object is missing a `macos` key, the property is assumed to be unsupported in the macOS SDK.
   1. If the release adds support for features other than properties, update [docs/style-spec/_generate/index.html](https://github.com/mapbox/mapbox-gl-js/blob/master/docs/style-spec/_generate/index.html).
   1. Run `yarn install && yarn run build-docs` to generate the style specification site.
   1. Commit these changes and open a PR in the mapbox-gl-js repository to get them reviewed and merged. (Keep going while you wait for a review.)
   1. Once the PR is merged, cherry-pick the changes into the mb-pages branch, so that they go live on mapbox.com ahead of the next Mapbox GL JS release.
1. Edit [this table](https://wiki.openstreetmap.org/wiki/Mapbox_GL#Features) at the OpenStreetMap Wiki to correctly indicate the status of any new features.

## Tell your friends!

_Takes under 5 minutes_

1. If this is your first time releasing the Mapbox macOS SDK:
   1. [Sign up for a CocoaPods trunk account](https://guides.cocoapods.org/making/getting-setup-with-trunk.html#getting-started).
   1. Get one of the [Mapbox-macOS-SDK pod](https://guides.cocoapods.org/making/getting-setup-with-trunk.html#adding-other-people-as-contributors)’s owners to [add you as an owner](https://cocoapods.org/pods/Mapbox-macOS-SDK).
1. Push the podspecs to CocoaPods trunk:
   ```bash
   pod trunk push platform/macos/Mapbox-macOS-SDK.podspec
   pod trunk push platform/macos/Mapbox-macOS-SDK-symbols.podspec
   ```
1. Copy `pod`’s success message and let Mapbox’s Mobile team know about the new release by sending this message to the `#mobile` Slack channel if you work for Mapbox or the `#gl-collab` Slack channel if you don’t.
1. Submit an update to the [OSM Software Watchlist](https://wambachers-osm.website/index.php/osm-software) to get a mention in the next issue of [WeeklyOSM](http://www.weeklyosm.eu/). ([Jinal](https://www.mapbox.com/about/team/jinal-foflia/) can help you submit the update.)

Whew! Now you can close the release’s [milestone](https://github.com/mapbox/mapbox-gl-native/milestones/) and create one for the next release!