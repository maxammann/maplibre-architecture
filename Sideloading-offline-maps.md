For some specific use cases, it may be desirable to [sideload](https://en.wikipedia.org/wiki/Sideloading) one or more offline packs with an application. Mapbox does not officially support the techniques below; they are described here for your convenience.

## iOS and macOS

### Requirements

A typical environment for developing native iOS or macOS applications.

### Generate the offline packs

Mapbox GL.app is an application that demonstrates the Mapbox Maps SDK for macOS. The iOS and macOS map SDKs are largely compatible with one another and use an identical offline storage format.

1. Go to [this repository’s releases page](https://github.com/mapbox/mapbox-gl-native/releases/), and look for the latest release of the Mapbox Maps SDK for macOS. macOS release tags begin with “macos-”.
1. Download and unzip Mapbox.GL.app.zip, then open Mapbox GL.app.
1. Choose the style using the View menu or the menu in the toolbar. Use View ‣ Custom Style to choose a custom style designed in Mapbox Studio.
1. Resize the window and adjust the map so that the extents of the desired offline pack are fully visible in the window.
1. Go to Window ‣ Offline Packs.
1. Click the + button. Enter a name for this offline pack, then click Add. Repeat this step for any offline pack you want to download.

Mapbox GL.app does not allow you to customize the offline packs other than choosing a name and zoom levels. If you need to accompany an offline pack with additional metadata, then you need to create a custom macOS application using the macOS SDK. Follow [the installation instructions](http://mapbox.github.io/mapbox-gl-native/macos/0.6.0/#installation), then consult the [`MGLOfflineStorage`](http://mapbox.github.io/mapbox-gl-native/macos/0.6.0/Classes/MGLOfflineStorage.html) documentation and/or [this fancy example](https://www.mapbox.com/ios-sdk/examples/offline-pack/) to build the offline downloading harness. When you create the offline pack, create an object and archive it:

```objc
// OfflineMapsViewController.m
NSDictionary *userInfo = @{
    @"name": @"Wapakoneta and Vicinity",
    @"sideLoaded": @YES,
};
NSData *context = [NSKeyedArchiver archivedDataWithRootObject:userInfo
                                        requiringSecureCoding:YES
                                                        error:NULL];
```

```swift
// OfflineMapsViewController.swift
let userInfo = [
    "name": "Wapakoneta and Vicinity",
    "sideLoaded": true,
]
let context = try! NSKeyedArchiver.archivedData(withRootObject: userInfo,
                                                requiringSecureCoding: true)
```

### Bundle the offline database

Once the offline packs finish downloading, the offline packs will be completely contained in the offline database file at the path below, along with any resources that may have been cached incidentally during your usage of Mapbox GL.app:

> ~/Library/Application Support/com.mapbox.MapboxGL/.mapbox/cache.db

Bundle this cache.db file with your application.

### Load the offline database

In your iOS or macOS application, download or copy the bundled `cache.db` to a temporary or other writeable location and [add the contents of the file](https://www.mapbox.com/ios-sdk/api/4.5.0/Classes/MGLOfflineStorage.html#/Adding%20Contents%20of%20File) to the offline storage. Use the `[MGLOfflineStorage addContentsOfFile:withCompletionHandler:]` or `[MGLOfflineStorage addContentsOfURL:withCompletionHandler:]` method to merge the offline database into the applications main map cache database.

```objc
// Copy the database to a temporary folder.
NSURL *databaseURL = [temporaryPathURL URLByAppendingPathComponent:@"cache.db"];
[[NSFileManager defaultManager] copyItemAtURL:sourceURL
                                        toURL:databaseURL
                                        error:NULL];

[[MGLOfflineStorage sharedOfflineStorage] addContentsOfURL:databaseURL withCompletionHandler:nil];
```

```swift
// Copy the database to a temporary folder.
let databaseURL = temporaryPathURL.appendingPathComponent("cache.db")
try! FileManager.default.copyItem(at: sourceURL, to: databaseURL)

MGLOfflineStorage.shared.addContents(of: databaseURL withCompletionHandler: nil)
```

</details>

Once the side-loaded database's contents are merged into the main database, they will be available for future requests.