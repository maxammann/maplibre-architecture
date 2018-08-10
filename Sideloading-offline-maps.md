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

Mapbox GL.app does not allow you to customize the offline packs other than choosing a name. If you need to accompany an offline pack with metadata, then you need to create a custom macOS application using the macOS SDK. Follow [the installation instructions](http://mapbox.github.io/mapbox-gl-native/macos/0.6.0/#installation), then consult the [`MGLOfflineStorage`](http://mapbox.github.io/mapbox-gl-native/macos/0.6.0/Classes/MGLOfflineStorage.html) documentation and/or [this fancy example](https://www.mapbox.com/ios-sdk/examples/offline-pack/) to build the offline downloading harness. When you create the offline pack, create an object and archive it:

```objc
NSDictionary *userInfo = @{ @"name": @"Wapakoneta and Vicinity", @"sideLoaded": @YES };
NSData *context = [NSKeyedArchiver archivedDataWithRootObject:userInfo];
```

### Bundle the offline database

Once the offline packs finish downloading, the offline packs will be completely contained in the offline database file at the path below, along with any resources that may have been cached incidentally during your usage of Mapbox GL.app:

> ~/Library/Application Support/com.mapbox.MapboxGL/.mapbox/cache.db

Bundle this cache.db file with your application.

### Load the offline database

In your iOS or macOS application, identify the portion of your code that runs before any `MGLMapView` is initialized and before the shared `MGLOfflineStorage` object is first invoked. This may be very early in your appication’s life cycle, especially if your user interface is managed by a storyboard. The most likely method would be `-[UIApplicationDelegate application:didFinishLaunchingWithOptions:]` (iOS) or `-[NSApplicationDelegate applicationDidFinishLaunching:]` (macOS). In this method, copy the bundled cache.db to this location ([source](https://github.com/mapbox/mapbox-gl-native/blob/fdc287ec3608850654196e3b3a682ca3c5039676/platform/darwin/src/MGLOfflineStorage.mm#L142-L169)):

> ~/Library/Application Support/_tld.app.bundle.id_/.mapbox/cache.db

where _tld.app.bundle.id_ is your application’s bundle identifier.

<details> 
<summary>Code sample (swift): get cache.db path </summary>
```swift
  do {
    let paths = NSSearchPathForDirectoriesInDomains(.applicationSupportDirectory, .userDomainMask, true)
    guard let bundleId = Bundle.main.bundleIdentifier else { return }
    let appSupportDirUrl = URL(fileURLWithPath: paths[0] + "/\(bundleId)")
    let mapboxDir = appSupportDirUrl.appendingPathComponent(".mapbox")
    try FileManager.default.createDirectory(at: mapboxDir, withIntermediateDirectories: true, attributes: nil);
    try FileManager.default.copyItem(at: URL(fileURLWithPath: "<path_to_your_cache.db>"), to: mapboxDir)
  } catch let error {
    print("Error: \(error.localizedDescription)");
  }
```
</details>


If you set a custom context on your sideloaded offline pack, for example to distinguish it from conventional offline packs, unarchive the `MGLOfflinePack.context` property’s value:

```objc
NSDictionary *userInfo = [NSKeyedUnarchiver unarchiveObjectWithData:pack.context];
BOOL wasSideLoaded = [userInfo[@"sideLoaded"] boolValue];
```

Note that our SDKs currently do not support sideloading at an arbitrary point after launch. There is only a small window of opportunity during which the offline packs can be sideloaded (in this rather kludgy way).