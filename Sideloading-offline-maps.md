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

In your iOS or macOS application, identify the portion of your code that runs before any `MGLMapView` is initialized and before the shared `MGLOfflineStorage` object is first invoked. This may be very early in your appication’s life cycle, especially if your user interface is managed by a storyboard. The most likely method would be `-[UIApplicationDelegate application:didFinishLaunchingWithOptions:]` (iOS) or `-[NSApplicationDelegate applicationDidFinishLaunching:]` (macOS). In this method, copy the bundled cache.db to this location ([source](https://github.com/mapbox/mapbox-gl-native/blob/fdc287ec3608850654196e3b3a682ca3c5039676/platform/darwin/src/MGLOfflineStorage.mm#L142-L169)):

> ~/Library/Application Support/_tld.app.bundle.id_/.mapbox/cache.db

where _tld.app.bundle.id_ is your application’s bundle identifier.

```objc
// AppDelegate.m

// Get the database’s current URL. This code assumes the database is bundled with the application.
NSURL *sourceURL = [[NSBundle mainBundle] URLForResource:@"cache"
                                           withExtension:@"db"];

// Get the destination folder URL. This code assumes your application has a bundle identifier set in Info.plist.
NSURL *appSupportURL = [[NSFileManager defaultManager] URLForDirectory:NSApplicationSupportDirectory
                                                              inDomain:NSUserDomainMask
                                                     appropriateForURL:nil
                                                                create:YES
                                                                 error:NULL];
NSString *bundleIdentifier = [[NSBundle main] bundleIdentifier];
NSURL *mapboxDirectoryURL = [[[NSURL fileURLWithPath:appSupportURL]
                              URLByAppendingPathComponent:bundleIdentifier isDirectory:YES]
                             URLByAppendingPathComponent:@".mapbox" isDirectory:YES];

// Create the destination folder if it doesn’t exist.
[[NSFileManager defaultManager] createDirectoryAtURL:mapboxDirectoryURL
                         withIntermediateDirectories:YES
                                          attributes:nil
                                               error:nil];

// Avoid backing up the offline cache onto iCloud, because it can be redownloaded.
[mapboxDirectoryURL setResourceValue:@YES
                              forKey:NSURLIsExcludedFromBackupKey
                               error:NULL];

// Copy the database to the destination folder.
NSURL *destinationURL = [mapboxDirectoryURL URLByAppendingPathComponent:@"cache.db"];
[[NSFileManager defaultManager] copyItemAtURL:sourceURL
                                        toURL:destinationURL
                                        error:NULL];
```

```swift
// AppDelegate.swift

// Get the database’s current URL. This code assumes the database is bundled with the application.
let sourceURL = Bundle.main.url(forResource: "cache", withExtension: "db")!

// Get the destination folder URL. This code assumes your application has a bundle identifier set in Info.plist.
let appSupportURL = try! FileManager.default.url(for: .applicationSupportDirectory,
                                                 in: .userDomainMask,
                                                 appropriateFor: nil,
                                                 create: true)
let bundleIdentifier = Bundle.main.bundleIdentifier!
let mapboxDirectoryURL = URL(fileURLWithPath: appSupportURL)
    .appendingPathComponent(bundleIdentifier, isDirectory: true)
    .appendingPathComponent(".mapbox", isDirectory: true)

// Create the destination folder if it doesn’t exist.
try! FileManager.default.createDirectory(at: mapboxDirectoryURL,
                                         withIntermediateDirectories: true,
                                         attributes: nil)

// Avoid backing up the offline cache onto iCloud, because it can be redownloaded.
try! (mapboxDirectoryURL as NSURL).setResourceValue(true, forKey: .isExcludedFromBackupKey)

// Copy the database to the destination folder.
let destinationURL = mapboxDirectoryURL.appendingPathComponent("cache.db")
try! FileManager.default.copyItem(at: sourceURL, to: destinationURL)
```

</details>


If you set a custom context on your sideloaded offline pack, for example to distinguish it from conventional offline packs, unarchive the `MGLOfflinePack.context` property’s value:

```objc
// AppDelegate.m
NSDictionary *userInfo = [NSKeyedUnarchiver unarchiveObjectOfClass:[NSDictionary class]
                                                          fromData:pack.context
                                                             error:NULL];
BOOL wasSideLoaded = [userInfo[@"sideLoaded"] boolValue];
```

```swift
// AppDelegate.swift
let userInfo = try! NSKeyedUnarchiver.unarchiveObject(ofClass: [String: Any].self,
                                                      from: pack.context)
let wasSideLoaded = userInfo?["sideLoaded"] as? Bool ?? false
```

Note that our SDKs currently do not support sideloading at an arbitrary point after launch. There is only a small window of opportunity during which the offline packs can be sideloaded (in this rather kludgy way).