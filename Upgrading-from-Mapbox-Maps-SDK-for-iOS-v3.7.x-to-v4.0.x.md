Mapbox Maps SDK for iOS v4.0.0 is the first major version of this SDK since 2015. To upgrade, follow [the installation instructions](https://www.mapbox.com/install/ios/), or run `pod update` with CocoaPods or `carthage update` with Carthage.

Version 4.0.0 contains [a number of important changes](https://www.mapbox.com/ios-sdk/api/4.0.0/). There are several backwards-incompatible changes to be aware of as you upgrade, which are discussed below.

## Runtime styling

We made a number of improvements to give you more control over the appearance of a map’s style. These improvements required us to make the following backwards-incompatible API changes to `MGLStyleLayer` and its subclasses:

* The layout and paint properties on subclasses of `MGLStyleLayer` are now of type `NSExpression` instead of `MGLStyleValue`. `MGLStyleValue` has been removed. [This guide](https://www.mapbox.com/ios-sdk/api/4.0.0/migrating-to-expressions.html) shows you how to migrate your runtime styling code to `NSExpression`. The “[Predicates and Expressions](https://www.mapbox.com/ios-sdk/api/4.0.0/predicates-and-expressions.html)” guide contains full details on how to write expressions that are compatible with style layers.
* A small number of `NSPredicate`s written for v3.7._x_ may need to cast feature attributes to the correct type, as described in [this section of the predicate guide](https://www.mapbox.com/ios-sdk/api/4.0.0/predicates-and-expressions.html#operands).

Additionally, `MGLRasterSource` was renamed `MGLRasterTileSource` and `MGLVectorSource` was renamed `MGLVectorTileSource`. Xcode automatically provides fix-it suggestions for these changes.

## Swift class methods

If your application is written in Swift, you’ll need to account for the following class methods being converted into class properties. Xcode automatically provides fix-it suggestions for these changes as well:

Old | New
----|----
`MGLStyle.streetsStyleURL()` etc. | `MGLStyle.streetsStyleURL` etc.
`MGLOfflineStorage.shared()` | `MGLOfflineStorage.shared`
`MGLAccountManager.setAccessToken("pk.…")` | `MGLAccountManager.accessToken = "pk.…"`

## Other changes

Any classes, methods, or properties that were deprecated in v3.7.6 are no longer available. See your project’s build errors for details about replacements.

The SDK no longer supports 32-bit simulators, such as the iPhone 5 and iPad 2 simulators. The SDK continues to support 32-bit devices, but note that the minimum iOS deployment version will increase to iOS 9.0 in a future release.

If you have any questions, please [contact Mapbox’s support team](https://www.mapbox.com/contact/support/).