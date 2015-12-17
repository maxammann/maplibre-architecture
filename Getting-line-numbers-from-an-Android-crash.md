Often when you encounter an Android native crash (ie one inside C++ not Java) or see a bug report with a log attached all you get is this from Android Studio's logcat:

```
10-30 10:19:20.848 24358-25130/com.mapbox.mapboxgl.testapp I/mbgl: {Map}[Sprite]: Can't find sprite named 'marsh-16'
10-30 10:19:21.988 24358-25130/com.mapbox.mapboxgl.testapp I/mbgl: {Map}[Android]: Not activating as we are not ready
10-30 10:19:22.008 24358-25130/com.mapbox.mapboxgl.testapp I/mbgl: {Map}[Android]: Not deactivating as we are not ready
10-30 10:19:22.078 24358-24551/com.mapbox.mapboxgl.testapp A/libc: Fatal signal 11 (SIGSEGV), code 1, fault addr 0x3c in tid 24551 (Tkj2y_5pUcsWvCQ)
```

The actual crash is the last line. But beyond that it is a segment violation at virtual memory address `0x3c` we have no useful details.

The first thing to fix this is to change the logcat filter. You want to set the far right box (probably set to "Show only selected application") to "No Filters". This will reveal all logs the entire Android system and other apps are spewing to logcat. Some phones (Samsung) are worse than others when it comes to logcat noise.

Next you want to narrow these logs to reveal the "tombstone" (Android speak for a crash report). I have found the easiest way it to type "`/DEBUG`" into the middle textbox (the one with the magnifying glass icon).

You should get something like this:
```
10-30 10:19:22.138 2982-2982/? E/DEBUG: unexpected waitpid response: n=24551, status=0000000b
10-30 10:19:22.138 2982-2982/? E/DEBUG: tid exited before attach completed: tid 24551
10-30 10:19:22.258 2969-3844/? E/: BitTube(): close sendFd (52)
10-30 10:19:22.258 2969-3844/? E/: BitTube(): close sendFd (53)
10-30 10:27:02.688 2982-2982/? I/DEBUG: *** *** *** *** *** *** *** *** *** *** *** *** *** *** *** ***
10-30 10:27:02.688 2982-2982/? I/DEBUG: Build fingerprint: 'samsung/zerofltedv/zeroflte:5.1.1/LMY47X/G920IDVU3DOJ6:user/release-keys'
10-30 10:27:02.688 2982-2982/? I/DEBUG: Revision: '11'
10-30 10:27:02.688 2982-2982/? I/DEBUG: ABI: 'arm'
10-30 10:27:02.688 2982-2982/? I/DEBUG: pid: 25808, tid: 27889, name: Tkj2y_5pUcsWvCQ  >>> com.mapbox.mapboxgl.testapp <<<
10-30 10:27:02.688 2982-2982/? I/DEBUG: signal 11 (SIGSEGV), code 1 (SEGV_MAPERR), fault addr 0x3c
10-30 10:27:02.688 2982-2982/? I/DEBUG:     r0 00000000  r1 00000000  r2 00000000  r3 0000591c
10-30 10:27:02.688 2982-2982/? I/DEBUG:     r4 dbb0b894  r5 d6537940  r6 dbb0b888  r7 dbb0b8c4
10-30 10:27:02.688 2982-2982/? I/DEBUG:     r8 dc50ca20  r9 fffff45c  sl 000000c8  fp dbb0b8b8
10-30 10:27:02.688 2982-2982/? I/DEBUG:     ip e1950d6c  sp dbb0b808  lr e16d386c  pc e182ecf8  cpsr 600e0010
10-30 10:27:02.688 2982-2982/? I/DEBUG:     #00 pc 00360cf8  /data/app/com.mapbox.mapboxgl.testapp-2/lib/arm/libmapbox-gl.so (uv_async_send+8)
10-30 10:27:02.688 2982-2982/? I/DEBUG:     #01 pc 00205868  /data/app/com.mapbox.mapboxgl.testapp-2/lib/arm/libmapbox-gl.so (mbgl::HTTPAndroidRequest::onResponse(int, std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> >, std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> >, std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> >, std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> >, std::__1::basic_string<char, std::_
10-30 10:27:02.688 2982-2982/? I/DEBUG:     #02 pc 002033e0  /data/app/com.mapbox.mapboxgl.testapp-2/lib/arm/libmapbox-gl.so (mbgl::nativeOnResponse(_JNIEnv*, _jobject*, long long, int, _jstring*, _jstring*, _jstring*, _jstring*, _jstring*, _jbyteArray*)+1620)
10-30 10:27:02.688 2982-2982/? I/DEBUG:     #03 pc 002798b7  /data/dalvik-cache/arm/data@app@com.mapbox.mapboxgl.testapp-2@base.apk@classes.dex
a
```

This reveals the full native stack trace. However it reports everything using program counter addresses (`#00 pc 00360cf8`) which does not mean much to us human types. And often the C++ symbols it does helpfully provide, are not so helpful:
`mbgl::HTTPAndroidRequest::onResponse(int, std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> >, std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> >, std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> >, std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> >, std::__1::basic_string<char, std::_`

To make sense of this tombstone, you need to use `ndk-stack` which is documented [here](http://developer.android.com/ndk/guides/ndk-stack.html). That page lists a few ways to get a tombstone into `ndk-stack` via file or piping.

However my favourite method is to simply select the lines from logcat (including the very important `*****...` line), paste into a text editor, and save as `crash.txt`.

As for the `ndk-stack` command, the two options it has are `-sym` and `-dump`.

First `-sym` is the path to the Android `.so` file. We want to use the pre-stripped binaries which you will find at `mapbox-gl-native/build/android-arm-v7/Release/lib.target/`. Note that `Release` can also be `Debug` and `android-arm-v7` can also be `android-x86-v7` depending on the `make` command you built the library with.

Quick note on stripped binaries: To save download size, Android apps strip the debug symbols from their JNI `.so` libraries before they are shipped. Our `make` does this when copying the `.so` to the `jni-libs` directory in Android Studio.

Next is `-dump`, this is simply the path to the `crash.txt` file you created.

So you end up with `ndk-stack -sym mapbox-gl-native/build/android-arm-v7/Release/lib.target/ -dump crash.txt`. If this command fails saying it can't find `ndk-stack` you need to ensure that the root directory of the Android NDK is on your path. (You will have to download this from [here](http://developer.android.com/ndk/downloads/index.html) if you don't have it installed)

The output from `ndk-stack` will be like:
```
********** Crash dump: **********
Build fingerprint: 'samsung/zerofltedv/zeroflte:5.1.1/LMY47X/G920IDVU3DOJ6:user/release-keys'
pid: 14778, tid: 15013, name: OkHttp https://  >>> com.mapbox.mapboxgl.testapp <<<
signal 11 (SIGSEGV), code 1 (SEGV_MAPERR), fault addr 0x3c
Stack frame #00 pc 00360cf8  /data/app/com.mapbox.mapboxgl.testapp-2/lib/arm/libmapbox-gl.so (uv_async_send+8): Routine uv_async_send at /home/travis/build/mapbox/mason/mason_packages/.build/libuv-1.7.5/src/unix/async.c:62
Stack frame #01 pc 00205868  /data/app/com.mapbox.mapboxgl.testapp-2/lib/arm/libmapbox-gl.so (mbgl::HTTPAndroidRequest::onResponse(int, std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> >, std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> >, std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> >, std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> >, std::__1::basic_string<char, std::_: Routine uv::async::send() at /home/leith/mb/mapbox-gl-native/build/android-arm-v7/../../src/mbgl/util/uv_detail.hpp:124
Stack frame #02 pc 002033e0  /data/app/com.mapbox.mapboxgl.testapp-2/lib/arm/libmapbox-gl.so (mbgl::nativeOnResponse(_JNIEnv*, _jobject*, long long, int, _jstring*, _jstring*, _jstring*, _jstring*, _jstring*, _jbyteArray*)+1620): Routine mbgl::nativeOnResponse(_JNIEnv*, _jobject*, long long, int, _jstring*, _jstring*, _jstring*, _jstring*, _jstring*, _jbyteArray*) at /home/leith/mb/mapbox-gl-native/build/android-arm-v7/../../platform/android/http_request_android.cpp:359 (discriminator 6)
Stack frame #03 pc 002798b7  /data/dalvik-cache/arm/data@app@com.mapbox.mapboxgl.testapp-2@base.apk@classes.dex
```

To understand this, lets take `(mbgl::HTTPAndroidRequest::onResponse(int, std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> >, std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> >, std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> >, std::__1::basic_string<char, std::__1::char_traits<char>, std::__1::allocator<char> >, std::__1::basic_string<char, std::_: Routine uv::async::send() at /home/leith/mb/mapbox-gl-native/build/android-arm-v7/../../src/mbgl/util/uv_detail.hpp:124` as an example. In this line we can see the function that crashed is `mbgl::HTTPAndroidRequest::onResponse` and the source file is `src/mbgl/util/uv_detail.hpp` on line 124.

This line is [here](https://github.com/mapbox/mapbox-gl-native/blob/master/src/mbgl/util/uv_detail.hpp#L124) which agrees with stack frame #00

Note that debug symbols and PC addresses are specific to the last `make` command you ran. This means you need to reproduce the crash locally if someone gives you a raw stack trace from another device/build/etc.

I find 90% of the time the line numbers from `ndk-stack` is enough to spot and fix programming errors in C++. To dig deeper currently requires a complex GDB setup which is documented [here](https://github.com/mapbox/mapbox-gl-native/wiki/Android-debugging-with-remote-GDB). However Google provide some interesting tips on using the Qt IDE in the video I linked to from [here](https://github.com/mapbox/mapbox-gl-native/issues/2755)

A older version of this guide is [here](https://github.com/mapbox/mapbox-gl-native/wiki/Symbolicating-Android-crashes).