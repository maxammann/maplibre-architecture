# Preparation

1. Install Android Studio, SDK and NDK, including build tools and platform tools.
2. Ensure `android-sdk/tools` and `android-sdk/platform-tools` are on your `PATH`
2. Build the app with: `BUILDTYPE=Debug make android`
3. Connect your device via USB
4. Check you can connect to the device by running `adb shell`. You should get the terminal from your device.
5. Exit with `exit`

# Extract system binaries from device
_(You will need to do this for every device and every Android OS version)_

1. Create a folder to hold the Android system binaries locally e.g. `~/android`
2. `cd` into the folder
3. Create a `system_lib` and `vendor_lib` folder
4. `cd system_lib`
5. `adb pull /system/lib`
6. `cd ../vendor_lib`
7. `adb pull /vendor/lib`. If you get permissions error you will need to get a list of each file and folder in `adb shell` then copy each file one at a time with `adb pull /vendor/lib/file`
8. `cd ..`
9. `adb pull /system/bin/app_process` (or on 64 bit phones `adb pull /system/bin/app_process32` and `adb pull /system/bin/app_process64`)
9. `adb pull /system/bin/linker`

# Install GDB server

1. Go to the NDK folder.
2. Copy `gdbserver` from `android-ndk/prebuilt/android-arm/gdbserver/gdbserver` to `mapbox-gl-native/android/java/MapboxGLAndroidSDK/src/main/jniLibs/armeabi-v7a/gdbserver.so`
**IMPORTANT** it must be renamed a .so file
3. Build and run the app in Android Studio
4. Android studio will copy and install the APK with gdbserver in it to your device

# Start the app paused

1. Open the project in Android Studio
2. Place a breakpoint in Java class `NativeMapView` constructor on `nativeCreate` line
3. Start app with Run -> Debug
4. Wait for app to start and hit breakpoint
5. Open up logcat and look for output from app
6. Note the process ID e.g. in `11-08 19:25:52.957  31834-31834/com.mapbox.mapboxgl.app V/FragmentActivity﹕ onCreate` it is `31834`

# Start gdbserver

1. Open a terminal
2. Run `adb forward tcp:5039 localfilesystem:/data/data/com.mapbox.mapboxgl.testapp/debug-pipe`
3. Run `adb shell run-as com.mapbox.mapboxgl.testapp /data/data/com.mapbox.mapboxgl.testapp/lib/gdbserver.so +debug-pipe --attach 31834`. Replace `31834` with the process ID from earlier.
_(You will need to do this each time you restart the app as the PID will change)_

You should see:
`Attached; pid = 31834
Listening on sockaddr socket debug-socket`

4. Leave the terminal open

# Start gdb

1. Open another terminal
2. Go to the NDK folder `android-ndk/toolchains/arm-linux-androideabi-4.9/prebuilt/linux-x86_64/bin`
- On OSX use `android-ndk/toolchains/arm-linux-androideabi-4.9/prebuilt/darwin-x86_64/bin`
3. Run `./arm-linux-androideabi-gdb ~/android/app_process`
4. `target remote :5039`
5. In GDB: `set solib-search-path ~/android/:~/android/system_lib:~/android/vendor_lib:~/android/vendor_lib/egl:~/path/to/mapbox-gl-native/build/android-arm-v7/Debug/lib.target/`
6. Check that all the debug symbols were loaded with `info sharedlibrary`
7. Check each .so has `Yes (*)` except for the last `libmapbox-gl.so` which must have only `Yes` i.e. (no star). If not double check your `solib-search-path`
8. `b jni.cpp:183` (the first line of `nativeCreate`)
9. `c`
10. Switch to Android Studio
11. Click Run -> Resume Program
12. Switch back to GDB. It should be paused at `nativeCreate`
13. GDB now has control, so `c` will continue execution (set breakpoints first)
**Note:**
If you encounter this crash:
```
Program received signal SIGILL, Illegal instruction.
0x7956a3a8 in _armv7_tick () from /home/leith/dev/mapbox-gl-native-mason/build/android/out/Debug/lib.target/libmapbox-gl.so
(gdb) bt
#0  0x7956a3a8 in _armv7_tick () from /home/leith/dev/mapbox-gl-native-mason/build/android/out/Debug/lib.target/libmapbox-gl.so
#1  0x795d1ccc in OPENSSL_cpuid_setup () from /home/leith/dev/mapbox-gl-native-mason/build/android/out/Debug/lib.target/libmapbox-gl.so
#2  0x400bd9c6 in ?? () from /home/leith/dev/android/linker
#3  0x400bda9e in ?? () from /home/leith/dev/android/linker
#4  0x400bdbf0 in ?? () from /home/leith/dev/android/linker
#5  0x400bdc6e in ?? () from /home/leith/dev/android/linker
#6  0x400bc1a6 in _start () from /home/leith/dev/android/linker
#7  0x41643c86 in dvmLoadNativeCode(char const*, Object*, char**) () from /home/leith/dev/android/system_lib/libdvm.so
#8  0x416600f4 in ?? () from /home/leith/dev/android/system_lib/libdvm.so
#9  0x41613ee8 in dvmJitToInterpNoChain () from /home/leith/dev/android/system_lib/libdvm.so
#10 0x41613ee8 in dvmJitToInterpNoChain () from /home/leith/dev/android/system_lib/libdvm.so
Backtrace stopped: previous frame identical to this frame (corrupt stack?)
```
You just need to `c` past it to the real crash. From https://bugs.launchpad.net/raspbian/+bug/1154042:
> Afaict openssl probes the capabilities of the user's CPU by trying to do things and trapping the illegal instruction errors. So a couple of sigills during startup is normal. When using a debugger in order to find the real failure in your application you must continue past the startup sigills.
14. Use GDB commands to debug

Read http://condor.depaul.edu/glancast/373class/docs/gdb.html for GDB commands