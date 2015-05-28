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

1. Download and decompress a zip of the latest iOS release at `http://mapbox.s3.amazonaws.com/mapbox-gl-native/ios/builds/mapbox-gl-ios-$VERSION.zip`, where `$VERSION` is the [latest version number](https://github.com/mapbox/mapbox-gl-native/releases), e.g. “0.2.15”. It contains a statically-linked `libMapboxGL.a`, a `MapboxGL.bundle` for resources, and a `Headers` folder.
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

> **Coming in beta 2:** In order to use Mapbox-hosted maps, you need to set the access token globally. Open the Info.plist file under the Supporting Files group. Select “Information Property List” and go to Editor ‣ Add Item. Set the Key to “MGLMapboxAccessToken” and the Value to the access token you retrieved from the Mapbox website.

### Using Interface Builder

The easiest way to get started is with a storyboard:

1. Drag a UIView out from the Object library into a `UIViewController` in a storyboard.
1. In the Identity inspector, set the custom class to “MGLMapView”.
1. In the Attributes inspector, set your access token _(beta 1 only)_. Optionally set a map ID, initial coordinates, and initial zoom level.  
1. _(Optional)_ Set up the `MGLMapView`’s Auto Layout constraints, and set its `viewControllerForLayoutGuides` outlet to the containing view controller.
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

If you need more customization options at launch, you can instantiate the `MGLMapView` programmatically instead of designing a storyboard. Insert the following lines into your view controller’s `-viewDidLoad`, substituting `<#your access token#>` with a string literal containing your access token:

```objc
// ViewController.m
#import "MapboxGL.h"

// …

- (void)viewDidLoad {
    [super viewDidLoad];
    MGLMapView *mapView = [[MGLMapView alloc] initWithFrame:self.view.bounds
                                                accessToken:<#your access token#>];
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
    let mapView = MGLMapView(frame: view.bounds, accessToken: <#your access token#>)
    mapView.autoresizingMask = .FlexibleWidth | .FlexibleHeight
    mapView.setCenterCoordinate(CLLocationCoordinate2D(latitude: -23.526, longitude: 148.162),
        zoomLevel: 15, animated: false)
    view.addSubview(mapView)
}
```

> **Coming in beta 2:** The `accessToken` parameter has been removed from `MGLMapView`’s initializers. To initialize an `MGLMapView`, call `-initWithFrame:`.

## Configuring Mapbox Metrics

By default, Mapbox GL sends anonymized location and usage data to Mapbox whenever the host app causes it to be gathered, via a fairly self-contained feature called Mapbox Metrics. The Mapbox Terms of Service require your app to provide users with a way to individually opt out of Mapbox Metrics. You can either add a setting to your app’s section in the Settings app (via a Settings.bundle in your app bundle) or integrate the setting directly into your app.  An [example implementation of a Settings app opt out](https://github.com/mapbox/mapbox-gl-native/tree/master/ios/app/Settings.bundle) is available as part of the iOS demo app.
