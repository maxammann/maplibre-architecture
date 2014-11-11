Google I/O videos (lots on how the UI renderer uses OpenGL etc):
https://www.youtube.com/user/GoogleDevelopers/playlists?shelf_id=6&view=50&sort=dd

A good resource to help you understand JNI:
http://developer.android.com/training/articles/perf-jni.html

The documentation for JNI (very handy):
http://docs.oracle.com/javase/7/docs/technotes/guides/jni/
http://docs.oracle.com/javase/7/docs/technotes/guides/jni/spec/jniTOC.html

Must reads for Android GL from NVIDIA (particularly the lifecycle stuff)
http://developer.download.nvidia.com/assets/mobile/files/AndroidLifecycleAppNote_v100.pdf
http://docs.nvidia.com/tegra/Content/How_To_header.html
http://docs.nvidia.com/gameworks/index.html#technologies/mobile/lifecycle_main.htm
http://docs.nvidia.com/gameworks/index.html#technologies/mobile/gles_egl_dev_notes.htm%3FTocPath%3DTechnologies%7CMobile%2520Technologies%7CMobile%2520How%2520Tos%7CDevelop%2520OpenGL%2520ES%25202.0%2520Applications%2520for%2520Tegra%7C_____2

Stuff from Android docs:
http://developer.android.com/guide/topics/graphics/opengl.html
http://developer.android.com/training/graphics/opengl/index.html
http://developer.android.com/reference/android/opengl/GLSurfaceView.html

One more thing, learn how to debug the native code in Android. It is very difficult and you will start swearing at Google but it was valuable in tracking down bugs.

Intel has a good overview (actually Intel have lots of good info on native Android):
https://software.intel.com/en-us/android/articles/how-to-test-and-debug-ndk-based-android-applications