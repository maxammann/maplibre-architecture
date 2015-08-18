1. Drag a `UIView` out from the Object library into a `UIViewController` in a storyboard.
1. In the Identity inspector, set the custom class to `MGLMapView`.
1. In the Attributes inspector, optionally set a map ID, initial coordinates, and initial zoom level.  
1. _(Optional)_ Set up the `MGLMapView`’s Auto Layout constraints.
1. Build and run.

![designable](https://cloud.githubusercontent.com/assets/1231218/6969674/3bc96d8a-d925-11e4-97a9-2bca4cf707f4.gif)

If your app needs to manipulate the map after launch, hook the `MGLMapView` up to an outlet:

1. Switch to the Assistant Editor.
1. Control-drag from the map view in Interface Builder to your view controller’s implementation.
1. Import Mapbox’s umbrella header into that file.

The resulting outlet declaration should look something like:

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
