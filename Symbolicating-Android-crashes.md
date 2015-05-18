* Make debug build: `BUILDTYPE=Debug ANDROID_ABI=x86 make android`.
* Reproduce crash
* Copy stack trace. It will look like:

  ```
********** Crash dump: **********
Build fingerprint: 'generic_x86/sdk_google_phone_x86/generic_x86:5.1/LKY45/1737576:eng/test-keys'
pid: 13072, tid: 13090, name: apboxgl.testapp  >>> com.mapbox.mapboxgl.testapp <<<
signal 11 (SIGSEGV), code 1 (SEGV_MAPERR), fault addr 0x0
Stack frame #00 pc 00000000  <unknown>
Stack frame #01 pc 0014f294  /data/app/com.mapbox.mapboxgl.testapp-1/lib/x86/libmapbox-gl.so (mbgl::Painter::render(mbgl::Style const&, mbgl::TransformState, std::__1::chrono::time_point<std::__1::chrono::steady_clock, std::__1::chrono::duration<long long, std::__1::ratio<1ll, 1000000000ll> > >)+2100)
Stack frame #02 pc 00129042  /data/app/com.mapbox.mapboxgl.testapp-1/lib/x86/libmapbox-gl.so (mbgl::MapContext::render()+562)
Stack frame #03 pc 00129a03  /data/app/com.mapbox.mapboxgl.testapp-1/lib/x86/libmapbox-gl.so
Stack frame #04 pc 000e03c4  /data/app/com.mapbox.mapboxgl.testapp-1/lib/x86/libmapbox-gl.so (mbgl::android::NativeMapView::invalidate(std::__1::function<void ()>)+132)
Stack frame #05 pc 00128bbd  /data/app/com.mapbox.mapboxgl.testapp-1/lib/x86/libmapbox-gl.so (mbgl::MapContext::update()+861)
Stack frame #06 pc 0012a223  /data/app/com.mapbox.mapboxgl.testapp-1/lib/x86/libmapbox-gl.so
Stack frame #07 pc 0012a0d9  /data/app/com.mapbox.mapboxgl.testapp-1/lib/x86/libmapbox-gl.so (uv::async::async_cb(uv_async_s*)+41)
Stack frame #08 pc 003608a1  /data/app/com.mapbox.mapboxgl.testapp-1/lib/x86/libmapbox-gl.so
Stack frame #09 pc 00360acc  /data/app/com.mapbox.mapboxgl.testapp-1/lib/x86/libmapbox-gl.so
Stack frame #10 pc 0036cedb  /data/app/com.mapbox.mapboxgl.testapp-1/lib/x86/libmapbox-gl.so
Stack frame #11 pc 00360f9b  /data/app/com.mapbox.mapboxgl.testapp-1/lib/x86/libmapbox-gl.so (uv_run+379)
Stack frame #12 pc 00126ac1  /data/app/com.mapbox.mapboxgl.testapp-1/lib/x86/libmapbox-gl.so (_ZZN4mbgl4util6ThreadINS_10MapContextEEC1IJRNS_4ViewERNS_10FileSourceERNS_7MapDataEEEERKNSt3__112basic_stringIcNSB_11char_traitsIcEENSB_9allocatorIcEEEENS0_14ThreadPriorityEDpOT_ENKUlvE_clEv+209)
Stack frame #13 pc 001269b3  /data/app/com.mapbox.mapboxgl.testapp-1/lib/x86/libmapbox-gl.so (_ZNSt3__114__thread_proxyINS_5tupleIJZN4mbgl4util6ThreadINS2_10MapContextEEC1IJRNS2_4ViewERNS2_10FileSourceERNS2_7MapDataEEEERKNS_12basic_stringIcNS_11char_traitsIcEENS_9allocatorIcEEEENS3_14ThreadPriorityEDpOT_EUlvE_EEEEEPvSS_+115)
Stack frame #14 pc 000211a8  /system/lib/libc.so (__pthread_start(void*)+56)
Stack frame #15 pc 0001c529  /system/lib/libc.so (__start_thread+25)
Stack frame #16 pc 000130f6  /system/lib/libc.so (__bionic_clone+70)
  ```
* Run `pbpaste | ndk-stack -sym build/android-x86/Debug/obj.target/android`. You should get output with file and line numbers, like:

  ```
********** Crash dump: **********
Build fingerprint: 'generic_x86/sdk_google_phone_x86/generic_x86:5.1/LKY45/1737576:eng/test-keys'
pid: 13072, tid: 13090, name: apboxgl.testapp  >>> com.mapbox.mapboxgl.testapp <<<
signal 11 (SIGSEGV), code 1 (SEGV_MAPERR), fault addr 0x0
Stack frame #00 pc 00000000  <unknown>
Stack frame #01 pc 0014f294  /data/app/com.mapbox.mapboxgl.testapp-1/lib/x86/libmapbox-gl.so (mbgl::Painter::render(mbgl::Style const&, mbgl::TransformState, std::__1::chrono::time_point<std::__1::chrono::steady_clock, std::__1::chrono::duration<long long, std::__1::ratio<1ll, 1000000000ll> > >)+2100): Routine operator() at /Users/john/Development/mapbox-gl-native/build/android-x86/../../src/mbgl/renderer/painter.cpp:266
Stack frame #02 pc 00129042  /data/app/com.mapbox.mapboxgl.testapp-1/lib/x86/libmapbox-gl.so (mbgl::MapContext::render()+562): Routine mbgl::MapContext::render() at /Users/john/Development/mapbox-gl-native/build/android-x86/../../src/mbgl/map/map_context.cpp:232
Stack frame #03 pc 00129a03  /data/app/com.mapbox.mapboxgl.testapp-1/lib/x86/libmapbox-gl.so: Routine operator() at /Users/john/Development/mapbox-gl-native/build/android-x86/../../src/mbgl/map/map_context.cpp:194
Stack frame #04 pc 000e03c4  /data/app/com.mapbox.mapboxgl.testapp-1/lib/x86/libmapbox-gl.so (mbgl::android::NativeMapView::invalidate(std::__1::function<void ()>)+132): Routine std::__1::function<void ()>::operator()() const at /Users/john/Development/mapbox-gl-native/mason_packages/osx-10.10/android-ndk/x86-9-r10d/bin/../lib/gcc/i686-linux-android/4.9/../../../../include/c++/4.9/functional:1756
Stack frame #05 pc 00128bbd  /data/app/com.mapbox.mapboxgl.testapp-1/lib/x86/libmapbox-gl.so (mbgl::MapContext::update()+861): Routine mbgl::MapContext::update() at /Users/john/Development/mapbox-gl-native/build/android-x86/../../src/mbgl/map/map_context.cpp:194
Stack frame #06 pc 0012a223  /data/app/com.mapbox.mapboxgl.testapp-1/lib/x86/libmapbox-gl.so: Routine operator() at /Users/john/Development/mapbox-gl-native/build/android-x86/../../src/mbgl/map/map_context.cpp:38
Stack frame #07 pc 0012a0d9  /data/app/com.mapbox.mapboxgl.testapp-1/lib/x86/libmapbox-gl.so (uv::async::async_cb(uv_async_s*)+41): Routine std::__1::function<void ()>::operator()() const at /Users/john/Development/mapbox-gl-native/mason_packages/osx-10.10/android-ndk/x86-9-r10d/bin/../lib/gcc/i686-linux-android/4.9/../../../../include/c++/4.9/functional:1756
Stack frame #08 pc 003608a1  /data/app/com.mapbox.mapboxgl.testapp-1/lib/x86/libmapbox-gl.so: Routine uv__async_event at /home/travis/build/mapbox/mason/mason_packages/.build/libuv-1.4.0/src/unix/async.c:89
Stack frame #09 pc 00360acc  /data/app/com.mapbox.mapboxgl.testapp-1/lib/x86/libmapbox-gl.so: Routine uv__async_io at /home/travis/build/mapbox/mason/mason_packages/.build/libuv-1.4.0/src/unix/async.c:165
Stack frame #10 pc 0036cedb  /data/app/com.mapbox.mapboxgl.testapp-1/lib/x86/libmapbox-gl.so: Routine uv__io_poll at /home/travis/build/mapbox/mason/mason_packages/.build/libuv-1.4.0/src/unix/linux-core.c:319
Stack frame #11 pc 00360f9b  /data/app/com.mapbox.mapboxgl.testapp-1/lib/x86/libmapbox-gl.so (uv_run+379): Routine uv_run at /home/travis/build/mapbox/mason/mason_packages/.build/libuv-1.4.0/src/unix/core.c:324
Stack frame #12 pc 00126ac1  /data/app/com.mapbox.mapboxgl.testapp-1/lib/x86/libmapbox-gl.so (_ZZN4mbgl4util6ThreadINS_10MapContextEEC1IJRNS_4ViewERNS_10FileSourceERNS_7MapDataEEEERKNSt3__112basic_stringIcNSB_11char_traitsIcEENSB_9allocatorIcEEEENS0_14ThreadPriorityEDpOT_ENKUlvE_clEv+209): Routine uv::loop::run() at /Users/john/Development/mapbox-gl-native/build/android-x86/../../src/mbgl/util/uv_detail.hpp:55
Stack frame #13 pc 001269b3  /data/app/com.mapbox.mapboxgl.testapp-1/lib/x86/libmapbox-gl.so (_ZNSt3__114__thread_proxyINS_5tupleIJZN4mbgl4util6ThreadINS2_10MapContextEEC1IJRNS2_4ViewERNS2_10FileSourceERNS2_7MapDataEEEERKNS_12basic_stringIcNS_11char_traitsIcEENS_9allocatorIcEEEENS3_14ThreadPriorityEDpOT_EUlvE_EEEEEPvSS_+115): Routine _ZNSt3__18__invokeIZN4mbgl4util6ThreadINS1_10MapContextEEC1IJRNS1_4ViewERNS1_10FileSourceERNS1_7MapDataEEEERKNS_12basic_stringIcNS_11char_traitsIcEENS_9allocatorIcEEEENS2_14ThreadPriorityEDpOT_EUlvE_JEEEDTclclsr3std3__1E7forwardIT_Efp_Espclsr3std3__1E7forwardIT0_Efp0_EEEOSQ_DpOSR_ at /Users/john/Development/mapbox-gl-native/mason_packages/osx-10.10/android-ndk/x86-9-r10d/bin/../lib/gcc/i686-linux-android/4.9/../../../../include/c++/4.9/__functional_base:413
Stack frame #14 pc 000211a8  /system/lib/libc.so (__pthread_start(void*)+56)
Stack frame #15 pc 0001c529  /system/lib/libc.so (__start_thread+25)
Stack frame #16 pc 000130f6  /system/lib/libc.so (__bionic_clone+70)
  ```

For more information: `open $ANDROID_NDK_PATH/docs/Programmers_Guide/html/md_3__key__topics__debugging__n_d_k-_s_t_a_c_k.htm`.