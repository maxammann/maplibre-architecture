Sideloading is a two-part process for making offline regions available to a mobile app. The first step is to package the tiles and resources necessary to create an offline region outside of an SDK. The second step is to merge the created offline package into the offline database of an SDK.

This document describes how to generate these offline packages and how to make them available to your mobile app.

## Android

#### How is sideloading different than downloading maps?"
 
Applications built with the <a href="https://docs.mapbox.com/android/maps/overview/">Mapbox Maps SDK for Android</a> can download maps of pre-selected regions for use when the device does not have network connectivity. This process is documented in detail in the <a href="https://www.mapbox.com/help/mobile-offline">Offline maps troubleshooting page</a>.

This system works well for smaller regions where you don’t have a large number of resources to download. (You can use the <a href="https://www.mapbox.com/help/offline-estimator/">offline tile count estimator</a> to understand the number of tiles in a specific region.) But, because tiles are downloaded individually, this approach can be too slow for larger regions and might result in poor user experience. In cases like this, offline sideloading can provide a better solution.

### Generate the offline package
To generate the offline package that contains the tiles and resources for a specific region, you have two options. You can either use a command-line tool, or you can use the macOS graphical interface. The CLI option is appropriate for any operating system, while the macOS option is only available in macOS environments.

Note that the extent of offline capabilities in your mobile app is limited to **6,000 tiles**. For more details on this "tile ceiling", see the [Limitations section](#limitations) of this documentation.

#### Use the command-line tool
Using a command-line tool is ideal if you want to build your offline packages server-side (for example, in a container) or as part of an automated system like CI in which a graphical interface is either not necessary or not desirable.

**Requirements**

- To compile the CLI, you need to set up your development environment as described in the [Building and developing from source](https://github.com/mapbox/mapbox-gl-native/blob/master/INSTALL.md) documentation.

##### CLI build instructions
1. Clone the Maps SDK: `git clone https://github.com/mapbox/mapbox-gl-native.git`
1. Change to the root folder: `cd mapbox-gl-native`
1. Check out the release commit equivalent to the version of the Maps SDK that you are using: ` git checkout {commit hash}` (view [all release commits](https://github.com/mapbox/mapbox-gl-native/releases))
1. Compile the binary: `make offline`

Once the build is complete, you'll see the message `Build Succeeded` in the terminal. The binary is available under the `build` folder (for example, on a Mac the file will be in `build/macos/Debug/mbgl-offline`).

##### Use the CLI
In the command line, move into to the folder where the `mbgl-offline` file is (`cd build/macos/Debug`), then run:
```
./mbgl-offline --north [north coordinate] --west [west coordinate] --south [south coordinate] --east [east coordinate] --minZoom [minimum zoom] --maxZoom [maximum zoom] --output [output file path] --style [style URL] --token [mapbox token]
```

The tiles that you requested will begin downloading.

**Example command:**
```
./mbgl-offline --north 71.44117085172385 --west -26.015625 --south 28.07198030177986 --east 28.916015625 --minZoom 4 --maxZoom 4 --output ~/europe.db --style mapbox://styles/developer/stylename --token developertoken
```
To see a few examples that you can reference when creating your region, take a look at `/bin/offline.sh`.

##### Optional flags
Use the following optional flags to customize your offline package. To see all available options, use the `--help` flag.

Flag | Description
--- | ---
`--style` | The map style URL.
`--north` | The northern-most coordinate in the bounding box.
`--south` | The southern-most coordinate in the bounding box.
`--east` | The eastern-most coordinate in the bounding box.
`--west` | The westernmost coordinate in the bounding box.
`--minZoom` | The minimum zoom level you want your region to have.
`--maxZoom` | The maximum zoom level you want your region to have.
`--pixelRatio` | Pixel ratio (or pixel density) is a device-dependent value provided by the OS. To see sample values for popular devices, see the [Material Design device metrics page](https://material.io/tools/devices/).
`--token` | The Mapbox access token to use for downloading tiles.
`--output` | The directory and file name you want your database to be called.
`--help` | See all available flag options.

_Warning:_ Take the available memory size into consideration when you choose your bounding coordinates and the `minZoom` and `maxZoom` levels.

#### Use the macOS graphical interface
Using the macOS graphical interface is a good approach if you create offline packages relatively infrequently if you prefer a visual tool to generate your packages or both.

This method of generating offline packages is only applicable to macOS users. For users with other operating systems, including Linux, use the CLI as described in the [Use the command-line tool ](#use-the-command-line-tool) section of this documentation.

**Requirements:**

To generate offline packages using the macOS graphical interface, you need a typical environment for developing native iOS or macOS applications.

##### Generate an offline package
1. Go to the [Mapbox GL Native repository’s release page](https://github.com/mapbox/mapbox-gl-native/releases/) and download the latest release of the Mapbox Maps SDK for macOS.
1. Unzip `Mapbox.GL.app.zip` and open `Mapbox GL.app`.
1. Choose the style using the _View_ menu or the menu in the toolbar. Use **View ‣ Custom Style** to choose a custom style designed in Mapbox Studio.
1. Resize the window and adjust the map so that the extents of the desired offline package are fully visible in the window.
1. Go to **Window ‣ Offline Packs**.
1. Click the **+** button to create a new offline package. Enter a name for this offline package, then click **Add**. Repeat this step for any offline package you want to download.

### Merge the offline package
Once you’ve created a new offline package, copy the file to the device so that the Offline Merging API can merge its contents into the main Maps SDK database. The file can be copied locally using `adb` or by downloading it from a server &mdash; both processes are described below. The Maps SDK loads all tiles and resources from one main database and provides the Offline Merging API to merge the contents of the new secondary database into the main one.

_Warning:_ Prior to v6.6.0 of the Maps SDK for Android, when the Offline Merging API was not available, the workaround for sideloading was to delete the main database file and add your own offline database. While this method is still technically possible, it is no longer recommended because it doesn't allow merging multiple databases and it is prone to crashing.

To merge a secondary database into the main Maps SDK database:

1. Move the secondary database onto the device. You have two options for doing so:
    - Physically copy it over USB using `adb`. For example, `adb push my-offline-region.db /path/to/app/files`.
    - Download the file from your server using a library like `OkHttp`.
2. In your code, call `OfflineManager.mergeOfflineRegions(String path, OfflineManager.MergeOfflineRegionsCallback callback)`.
    - `path` provides the secondary database (writable) path. If the app’s process doesn’t have write-access to the provided path, the file will be copied to the temporary, internal directory for the duration of the merge. (The secondary database may need to be upgraded to the latest schema. This is done in-place and requires write-access to the provided path.)
    - `callback` is the completion/error callback. When the merge is completed or fails, the `OfflineManager.MergeOfflineRegionsCallback` will be invoked on the main thread.

For a complete example, take a look at [`MergeOfflineRegionsActivity`](https://github.com/mapbox/mapbox-gl-native/blob/android-v{{constants.MAP_SDK_VERSION}}/platform/android/MapboxGLAndroidSDKTestApp/src/main/java/com/mapbox/mapboxsdk/testapp/activity/offline/MergeOfflineRegionsActivity.kt) and the [sample database](https://github.com/mapbox/mapbox-gl-native/blob/master/platform/android/MapboxGLAndroidSDKTestApp/src/main/assets/offline_test.db). For further details, see the [`mergeOfflineRegions` documentation](https://docs.mapbox.com/android/api/map-sdk/{{constants.MAP_SDK_VERSION}}/com/mapbox/mapboxsdk/offline/OfflineManager.html#mergeOfflineRegions-java.lang.String-com.mapbox.mapboxsdk.offline.OfflineManager.MergeOfflineRegionsCallback-).

### Database location on Android
On Android, you can move the location of the main Maps SDK database from internal storage to external storage. To do so, make sure the application has the `WRITE_EXTERNAL_STORAGE` permission inside the `AndroidManifest.xml` and the following flag:
```
<application>
  <meta-data
    android:name="com.mapbox.SetStorageExternal"
    android:value="true" />
    ...
</application>
```


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

In your iOS or macOS application, download or copy the bundled `cache.db` to a temporary or other writeable location and [add the contents of the file](https://www.mapbox.com/ios-sdk/api/4.5.0/Classes/MGLOfflineStorage.html#/Adding%20Contents%20of%20File) to the offline storage. Use the `-[MGLOfflineStorage addContentsOfFile:withCompletionHandler:]` or `-[MGLOfflineStorage addContentsOfURL:withCompletionHandler:]` method to merge the offline database into the applications main map cache database.

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

MGLOfflineStorage.shared.addContents(of: databaseURL, withCompletionHandler: nil)
```

</details>

Once the side-loaded database's contents are merged into the main database, they will be available for future requests.