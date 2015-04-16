## Installing the framework

There are two supported methods for installing Mapbox GL into your iOS project: via [CocoaPods](https://cocoapods.org/), or manually within Xcode.

### Using CocoaPods (recommended)

Your app's `Podfile` should be as follows (plus normal lines for any other pods used):

```ruby
pod 'MapboxGL', :podspec => 'https://raw.githubusercontent.com/mapbox/mapbox-gl-native/master/ios/MapboxGL.podspec'

use_frameworks!
```

Then, to update to the latest published version:

```bash
$ pod update
```

There is no way to pin versions right now; that will come when we release officially and on the CocoaPods master repository.

Now you’re ready to [instantiate an `MGLMapView`](#instantiating-an-mglmapview) and [configure Mapbox Metrics](#configuring-mapbox-metrics).

### Using Xcode

1. Download and decompress a zip or tarball of the [latest iOS release](https://github.com/mapbox/mapbox-gl-native/releases). It contains a statically-linked `libMapboxGL.a`, a `MapboxGL.bundle` for resources, and a `Headers` folder.
2. Drag these three items into the Project navigator of your project. When prompted, check the “Copy items if needed” box. Ensure that Xcode has made the following changes to your project:
   - `Headers` is in your *Header Search Paths* (`HEADER_SEARCH_PATHS`) build setting.
   - `MapboxGL.bundle` is in your target's *Copy Bundle Resources* build phase.
   - `libMapboxGL.a` is in your target's *Link Binary With Libraries* build phase.
3. Add the following Cocoa framework dependencies to your target's *Link Binary With Libraries* build phase:
   - `CoreTelephony.framework`
   - `GLKit.framework`
   - `ImageIO.framework`
   - `MobileCoreServices.framework`
   - `QuartzCore.framework`
   - `SystemConfiguration.framework`
   - `libc++.dylib`
   - `libsqlite3.dylib`
   - `libz.dylib`
4. Add `-ObjC` to your target's "Other Linker Flags" build setting (`OTHER_LDFLAGS`).

Now you’re ready to [instantiate an `MGLMapView`](#instantiating-an-mglmapview) and [configure Mapbox Metrics](#configuring-mapbox-metrics).

## Instantiating an `MGLMapView`

To use any Mapbox-hosted map style, including a style bundled with Mapbox GL, you need a Mapbox access token. Log into your Mapbox account and [grab an access token](https://www.mapbox.com/account/apps/).

### Using Interface Builder

The easiest way to get started is with a storyboard:

1. Drag a UIView out from the Object library into a `UIViewController` in a storyboard.
1. In the Identity inspector, set the custom class to “MGLMapView”.
1. In the Attributes inspector, set your access token. Optionally set a map ID, initial coordinates, and initial zoom level.
1. Build and run.

![designable](https://cloud.githubusercontent.com/assets/1231218/6969674/3bc96d8a-d925-11e4-97a9-2bca4cf707f4.gif)

If you need to interact with the map programmatically after launch, hook the `MGLMapView` up to an outlet by Control-dragging from the map view in Interface Builder to your view controller’s implementation, and import Mapbox GL’s umbrella header into that file. The resulting outlet declaration should look something like:

```objc
// ViewController.m
#import "MapboxGL.h"

@interface ViewController : UIViewController

@property (strong) IBOutlet MGLMapView *mapView;

@end
```

```swift
// ViewController.swift
import MapboxGL

class ViewController: UIViewController {
    @IBOutlet var mapView: MGLMapView!
}
```

### In code

If you need more customization options at launch, you can instantiate the `MGLMapView` programmatically instead of designing a storyboard. Insert the following lines into your view controller’s `-viewDidLoad`, substituting your access token:

```objc
// ViewController.m
#import "MapboxGL.h"

// …

- (void)viewDidLoad {
    [super viewDidLoad];
    MGLMapView *mapView = [[MGLMapView alloc] initWithFrame:[[self view] bounds]
                                                accessToken:@"my.accessToken1234567890"];
    mapView.autoresizingMask = UIViewAutoresizingFlexibleWidth | UIViewAutoresizingFlexibleHeight;
    [mapView setCenterCoordinate:CLLocationCoordinate2DMake(-23.526, 148.162)
                       zoomLevel:15
                        animated:NO];
    [self.view addSubview:mapView];
}
```

```swift
// ViewController.swift
import MapboxGL

// …

override func viewDidLoad() {
    super.viewDidLoad()
    let mapView = MGLMapView(frame: view.bounds, accessToken: "my.accessToken1234567890")
    mapView.autoresizingMask = .FlexibleWidth | .FlexibleHeight
    mapView.setCenterCoordinate(CLLocationCoordinate2D(latitude: -23.526, longitude: 148.162),
        zoomLevel: 15, animated: false)
    view.addSubview(mapView)
}
```

## Configuring Mapbox Metrics

By default, Mapbox GL sends anonymized location and usage data to Mapbox whenever the host app causes it to be gathered, via a fairly self-contained feature called Mapbox Metrics. The Mapbox Terms of Service require your app to provide users with a way to individually opt out of Mapbox Metrics. You can either add a setting to your app’s section in the Settings app or integrate the setting directly into your app.  An [example implementation of a Settings app opt out](https://github.com/mapbox/mapbox-gl-native/tree/master/ios/app/Settings.bundle) is available as part of the iOS demo app.

To conserve your users’ battery life, you can temporarily disable Mapbox Metrics while your app is in the background state and where it isn’t otherwise gathering location data. For example, if `UIBackgroundModes` contains `location`, the app is in the background, and none of the `CLLocationManager`s your app manages is active, call `-[MGLMapboxEvents pauseMetricsCollection]`. When your code resumes listening for location updates via a `CLLocationManager`, call `-[MGLMapboxEvents resumeMetricsCollection]`.