### 1. Load in the map
To ensure no network issues interfere with our performance testing, we'll use the offline map activity in the testapp. download a region for testing and make sure to remain in the area once finished.

### 2. Reset the frame stats data
Use `adb shell dumpsys gfxinfo com.mapbox.mapboxsdk.testapp reset` to reset the frame stats data.

### 3. Perform gestures
Perform the map gestures you want to test performance of making sure a sufficient number of frames are captured.

### 4. Retrieve the frame stats
Retrieve the frame stats using `adb shell dumpsys gfxinfo com.mapbox.mapboxsdk.testapp framestats`.

What the results look like graphed:
![](https://cloud.githubusercontent.com/assets/1081815/14079823/50aa7fea-f509-11e5-9290-6490b1143478.png)
