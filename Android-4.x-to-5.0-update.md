# Upgrade from 4.x

We've been hard at work making our maps SDK work even better than previous versions. Both performance improvements and new features have been added to the SDK. This meant we had to change up some of the APIs you were using. This document walks through the steps to take when making the upgrade.

### Manifest
The telemetry service package has moved. You'll need to change this from:

```xml
<service android:name="com.mapbox.mapboxsdk.telemetry.TelemetryService"/>
```
to:
```xml
<service android:name="com.mapbox.services.android.telemetry.service.TelemetryService"/>
```


### Adding new lifecycles

Part of supporting Android Nougat means we have to also support the new Multi-Window feature. This required us to move some of the logic to the `onStart()` and `onStop()` methods. When switching over to 5.0 Android Studio won't complain about these missing but when compiling your application will immediately crash if you don't add them to the activity:

```java
@Override
protected void onStart() {
  super.onStart();
  mapView.onStart();
}

@Override
protected void onStop() {
  super.onStop();
  mapView.onStop();
}
```

**Note:** These new map view lifecycles are in addition to all the ones required in 4.x

### Mapbox attribution name changes

We've made significant improvements to our attribution naming. All attributes now start with `mapbox_` for easier identifying and removing any potential conflicts with other libraries. In addition, many of the attributes are now named after their Java counterpart. Instead of listing all the changes that have occurred, you might find the [`attrs.xml`](https://github.com/mapbox/mapbox-gl-native/blob/master/platform/android/MapboxGLAndroidSDK/src/main/res/values/attrs.xml) file which has all the names. Some of the common attributes you might use in your XML map view are given in the table below:

| 4.x attribute names | 5.0 attribute names      |
| ------------------- |:------------------------:|
| `center_latitude`   | `mapbox_cameraTargetLat` |
| `center_longitude`  | `mapbox_cameraTargetLng` |
| `zoom`              | `mapbox_cameraZoom`      |
| `direction`         | `mapbox_cameraBearing`   |
| `tilt`              | `mapbox_cameraTilt`      |
| `style_url`         | `mapbox_styleUrl`        |
| `zoom_max`          | `mapbox_cameraZoomMax`   |
| `zoom_min`          | `mapbox_cameraZoomMin`   |
| `rotate_enabled`    | `mapbox_uiRotateGestures`|

Note that the `access_token` attribution doesn't exist anymore, more on this in the next sections.

#### Style string name changes

You can still access the default map style urls by using `mapbox:mapbox_styleUrl="@string/` but the string id's have changed to include `mapbox_` in front of them. For example, `@string/style_light` now becomes `@string/mapbox_style_light`.

### Mapbox access token

There is now only one method where you should be setting your access token. If you were previously setting the access token in XML, this is no longer an option. in 4.x you'd set your access token through the `MapboxAccountManager` object which could either belong in the application class (preferred) or in your activities class before `setContentView` is called. To set the access token now, see the example below:

```java
Mapbox.getInstance(this, "<Access token here>");
```

If you want to get the access token to pass into another API (common when using mapbox-java) you can get it with:

```java
Mapbox.getAccessToken()
```

### Other methods you might be using

| 4.x method names                                              | 5.0 method names                      |
| ------------------------------------------------------------- |:-------------------------------------:|
| `MapboxMap.getMaxZoom()`                                      | `MapboxMap.getMaxZoomLevel()`         |
| `mapView.getStyleUrl()`                                       | `mapboxMap.getStyleUrl()`             |
| `map.getMarkerViewManager().scheduleViewMarkerInvalidation()` | `map.getMarkerViewManager().update()` |
