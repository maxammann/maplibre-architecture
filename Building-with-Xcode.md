## Building with Xcode

Before getting started, make sure you've [[installed all prerequisites for building Mapbox GL|Prerequisites]]

To use Xcode, you'll first need to generate a project based on the [[CMake]] files. To do so, open a Terminal window, and run one of these commands [[from your checkout|Code]] directory.

Run `make xproj` to create a project for building for **macOS**.  
Run `make iproj` to create a project targeting **iOS**.  
Run `make qtproj` to create a project targeting **Qt**.

This will generate the Xcode project files in distinct directories in the `build` folder. You can use these workspaces side by side. You should see output along these lines:

```
$ make xproj
mkdir -p build/macos
(cd build/macos && cmake -G Xcode ../..)
-- The CXX compiler identification is AppleClang 9.0.0.9000039
-- The C compiler identification is AppleClang 9.0.0.9000039
-- Check for working CXX compiler: /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang++
-- Check for working CXX compiler: /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Check for working C compiler: /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang
-- Check for working C compiler: /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Updating submodules...
-- Downloading: https://nodejs.org/download/release/v6.10.3/SHASUMS256.txt
-- NodeJS: Using node, version v6.10.3
-- Downloading: https://nodejs.org/download/release/v6.10.3/node-v6.10.3-headers.tar.gz
-- Configuring done
-- Generating done
-- Build files have been written to: [...]/build/macos
open platform/macos/macos.xcworkspace
```

On subsequent runs of `make *proj`, you'll see much shorter blurbs, or nothing at all (it'll just open the project in Xcode) when there were no changes in the CMake files.

## Workspace structure

## 
