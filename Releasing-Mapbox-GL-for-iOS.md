## GitHub

1. Choose a version number per [Semantic Versioning](http://semver.org/). Let's call it _x_._y_._z_.
1. If necessary, update [the screenshot](https://github.com/mapbox/mapbox-gl-native/blob/master/ios/screenshot.png).
1. Create a tag ios-v<i>x</i>._y_._z_ and push the tag. 

## CocoaPods

```bash
version=$(perl -ne "/m.version\s*=\s*'(.*?)'/ && print \$1" ios/MapboxGL.podspec)
make clean
./scripts/package_ios.sh
cd build/ios/pkg/static/
rm -f ../mapbox-gl-ios-$version.zip
zip -r ../mapbox-gl-ios-$version.zip *
cd -
mapbox auth production ********
aws s3 cp --acl public-read build/ios/pkg/mapbox-gl-ios-$version.zip s3://mapbox/mapbox-gl-native/ios/mapbox-gl-ios-$version.zip
```