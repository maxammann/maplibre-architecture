This document describes a few strategies that can be used to reduce the resulting APK size when using the Mapbox Android SDK.

### 1. Use ProGuard or DexGuard

[ProGuard](https://developer.android.com/studio/build/shrink-code.html) and [DexGuard](https://www.guardsquare.com/en/dexguard), even without obfuscation, will remove unused Java code from your code and its dependencies.

### 2. Drop architectures you don't need

The Mapbox SDK ships with 6 architectures:

```
./arm64-v8a/libmapbox-gl.so
./armeabi/libmapbox-gl.so
./armeabi-v7a/libmapbox-gl.so
./mips/libmapbox-gl.so
./x86/libmapbox-gl.so
./x86_64/libmapbox-gl.so
```

Each of these files add up to the resulting APK. If, for example, your app doesn't need x86 support, you could drop `x86` and `x86_64` and save some space. See "ABI splitting" below for details.

### 3. APK splitting

This is a feature that lets you build an APK file for each CPU, only containing the relevant native libraries. This process is described in the [Android Studio Project Site](http://tools.android.com/tech-docs/new-build-system/user-guide/apk-splits#TOC-ABIs-Splits).

If you distribute your app via Google Play, you can benefit from this approach through the [Multiple APK Support](https://developer.android.com/google/play/publishing/multiple-apks.html) distribution feature.

#### Sample code

Mapbox publishes a [Demo App](https://play.google.com/store/apps/details?id=com.mapbox.mapboxandroiddemo) to the Google Play store that showcases core SDK features.

This app is [open source](https://github.com/mapbox/mapbox-android-demo) and benefits from APK splitting for smaller binary distribution. You can learn how we do this in the [`build.gradle`](https://github.com/mapbox/mapbox-android-demo/blob/master/MapboxAndroidDemo/build.gradle) file.

### Next

The Mapbox team is actively looking at other ways to reduce the SDK size ([for example](https://github.com/mapbox/mapbox-gl-native/issues/5656)).

If you have questions or ideas that you'd like to share, please [get in touch with us](https://github.com/mapbox/mapbox-gl-native/issues/new).