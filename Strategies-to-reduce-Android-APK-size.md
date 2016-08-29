Currently, the Mapbox Android SDK v4.1 adds about 11MB to your app. This document describes a few strategies that can be used to reduce the resulting APK size.

### Use ProGuard

[ProGuard](https://developer.android.com/studio/build/shrink-code.html), even without obfuscation, will remove unused Java code from your code and its dependencies.

### Drop architectures you don't need

The Mapbox SDK ships with 6 architectures:

```
./arm64-v8a/libmapbox-gl.so
./armeabi/libmapbox-gl.so
./armeabi-v7a/libmapbox-gl.so
./mips/libmapbox-gl.so
./x86/libmapbox-gl.so
./x86_64/libmapbox-gl.so
```

Each of these files add about 1.7-1.9 MB to the resulting APK. If, for example, your app doesn't need x86 support, you could drop `x86` and `x86_64` and save about 3.6MB. See "ABI splitting" below for details.

### ABI splitting

This is a feature that lets you build an APK file for each CPU, only containing the relevant native libraries. This process is described in the [Android Studio Project Site](http://tools.android.com/tech-docs/new-build-system/user-guide/apk-splits#TOC-ABIs-Splits).

If you distribute your app via Google Play, you can benefit from this approach through the [Multiple APK Support](https://developer.android.com/google/play/publishing/multiple-apks.html) distribution feature.

### Next

The Mapbox team is actively looking at other ways to reduce the SDK size. Some examples are:
* https://github.com/mapbox/mapbox-gl-native/issues/5656
* https://github.com/mapbox/mapbox-gl-native/issues/2354
* https://github.com/mapbox/mapbox-gl-native/issues/2355

If you have questions or any other ideas, please get in touch with us.