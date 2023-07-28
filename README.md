![not-maintenace](https://img.shields.io/maintenance/yes/2018)

No Maintenance Intended

---

# Image Detection Plugin (Android & iOS)

This plugin allows the application to detect if an inputed image target is visible, or not, by matching the image features with the device camera features using [OpenCV](http://opencv.org/) (v3.1. on Android, v2.4.13 on iOS)  It also presents the device camera preview in the background.

## cordova-android@7.0.0

The branch [cordova@7.0.0](https://github.com/Cloudoki/ImageDetectionCordovaPlugin/tree/cordova%407.0.0) should work using this version.

## Changes

- Added setting multiple patterns and loop functionality to detect which is visible

### Note (view mode)

The plugin is aimed to work in **portrait mode**, should also work in landscape but no guarantees.

## Install

To install the plugin in your current Cordova project run outside you project root

```shell
git clone https://github.com/Cloudoki/ImageDetectionCordovaPlugin.git
cd <your-project-root>
cordova plugin add ../ImageDetectionCordovaPlugin
```

### Android

- The plugin aims to be used with Android API >= 16 (4.1 Jelly Bean).

### IOS

- The plugin aims to be used with iOS version >= 7.
- **Important!** Go into src/ios folder and extract opencv2.framework from the zip file into the same folder.
- Since iOS 10, `<key>NSCameraUsageDescription</key>` is required in the project Info.plist of any app that wants to use Camera.

The plugin should add this automatically but in case this does not happen to add it, just open the project in XCode, go to the Info tab and add the `NSCameraUsageDescription` key with a string value that explain why your app need an access to the camera.

### Note

In *config.xml* add Android and iOS target preference

```javascript
<platform name="android">
    <preference name="android-minSdkVersion" value="16" />
</platform>
<platform name="ios">
    <preference name="target-device" value="handset"/>
    <preference name="deployment-target" value="7.0"/>
</platform>
```

And don't forget to set the background to be transparent or the preview may not shown up.
Again in *config.xml* add the following preference.

```javascript
<preference name="backgroundColor" value="0x00000000" />
```

## Usage

The plugin offers the functions `startProcessing`, `setDetectionTimeout`, `isDetecting` and `setPattern`.

**`startProcessing`** - the plugin will process the video frames captured by the camera if the inputed argument is `true`, if the argument is `false` no frames will be processed. Calls on success if the argument is set and on error if no value set.

**Note:** the plugins start with this option true.

```javascript
startProcessing(true or false, successCallback, errorCallback);
```

**`isDetecting`** - the plugin will callback on success function if detecting the pattern or on error function if it's not. The response will also say what index of the patterns is being detected, the detection rect coordinates and the center/centroid in a JSON object. Just parse it using `JSON.parse()`.

```javascript
isDetecting(successCallback, errorCallback);
```

```json
// JSON RESPONSE EXAMPLE
{
  "message": "pattern detected",
  "index": 0,
  "coords": {
    "1": { "x": 170.839401, "y": 181.510239 },
    "2": { "x": 293.745239, "y": 180.525345 },
    "3": { "x": 301.409363, "y": 352.518280 },
    "4": { "x": 171.494492, "y": 360.808655 }
  },
  "center": {
    "x": 234.956223, "y": 268.231842
  }
}
```

**`setDetectionTimeout`** - this function will set a timeout (**in seconds**) in which the processing of the frames will not occur. Calls on success if the argument is set and on error if no value set.

```javascript
setDetectionTimeout(timeout, successCallback, errorCallback);
```

**`setPatterns`** - sets the patterns targets to be detected. Calls on success if the patterns are set and on error if one or more patterns fail to be set. The input patterns must be an array of base64 image string.

```javascript
setPatterns([base64image, ...], successCallback, errorCallback);
```

## Usage example

```javascript
var imgDetectionPlugin = window.plugins.ImageDetectionPlugin || new ImageDetectionPlugin();

imgDetectionPlugin.startProcessing(true, function(success){console.log(success);}, function(error){console.log(error);});

imgDetectionPlugin.isDetecting(function(success){
  console.log(success);
  var resp = JSON.parse(success);
  console.log(resp.index, "image detected - ", indexes[resp.index], resp.coords, resp.center);
}, function(error){console.log(error);});

function setAllPatterns(patterns) {
  imgDetectionPlugin.setPatterns(patterns, function(success){console.log(success);}, function(error){console.log(error);});
}

var loadAllImg = 0;
var patternsHolder = [];
var indexes = {};
var limit = 3;

function ToDataURL (self) {
  var canvas = document.createElement('canvas');
  var ctx = canvas.getContext('2d');
  var dataURL;
  canvas.height = self.height;
  canvas.width = self.width;
  ctx.drawImage(self, 0, 0);
  dataURL = canvas.toDataURL("image/jpeg", 0.8);
  patternsHolder.push(dataURL);
  indexes[loadAllImg] = self.src.substr(self.src.lastIndexOf("/") + 1);
  loadAllImg += 1;
  console.log("!!!", loadAllImg, indexes);
  if(loadAllImg == limit){
    console.log("patterns set", patternsHolder);
    setAllPatterns(patternsHolder);
  }
  canvas = null;
}

var img = new Image();
img.crossOrigin = "Anonymous";
img.onload = function(){
  ToDataURL(this)
};
img.src = "img/patterns/target1.jpg";

var img = new Image();
img.crossOrigin = "Anonymous";
img.onload = function(){
  ToDataURL(this)
};
img.src = "img/patterns/target2.jpg";

var img = new Image();
img.crossOrigin = "Anonymous";
img.onload = function(){
  ToDataURL(this)
};
img.src = "img/patterns/target3.jpg";

imgDetectionPlugin.setDetectionTimeout(2, function(success){console.log(success);}, function(error){console.log(error);});
```

## Demo Project

[ImageDetectionDemoApp](https://github.com/a31859/ImageDetectionDemoApp)
