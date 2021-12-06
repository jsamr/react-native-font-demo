## Goal

Be able to use font styles modifiers such as `weight` and `style` in combination with a custom font family, in both iOS and Android.

```jsx
<Text style={{ fontFamily: "Raleway", fontWeight: "100", style: "italic" }}>Hello world!</Text>
<Text style={{ fontFamily: "Raleway", fontWeight: "bold", style: "normal" }}>Hello world!</Text>
```

### Prerequisites

#### Assets

You need the [whole Raleway font family](https://fonts.google.com/share?selection.family=Raleway:ital,wght@0,100;0,200;0,300;0,400;0,500;0,600;0,700;0,800;0,900;1,100;1,200;1,300;1,400;1,500;1,600;1,700;1,800;1,900), extracted in a temporary folder. That is:

- `Raleway-Thin.ttf` (100)
- `Raleway-ThinItalic.ttf`
- `Raleway-ExtraLight.ttf` (200)
- `Raleway-ExtraLightItalic.ttf`
- `Raleway-Light.ttf` (300)
- `Raleway-LightItalic.ttf`
- `Raleway-Regular.ttf` (400)
- `Raleway-Italic.ttf`
- `Raleway-Medium.ttf` (500)
- `Raleway-MediumItalic.ttf`
- `Raleway-SemiBold.ttf` (600)
- `Raleway-SemiBoldItalic.ttf`
- `Raleway-Bold.ttf` (700)
- `Raleway-BoldItalic.ttf`
- `Raleway-ExtraBold.ttf` (800)
- `Raleway-ExtraBoldItalic.ttf`
- `Raleway-Black.ttf` (900)
- `Raleway-BlackItalic.ttf`

We will assume those files are now stored in `/tmp/raleway/`

#### Setup

```sh
react-native init FontDemo
cd FontDemo
```

#### Android

For Android, we are going to use [XML Fonts](https://developer.android.com/guide/topics/ui/look-and-feel/fonts-in-xml) to define variants of a base font family.

> **Remark**:
> There has been an attempt to provide enhanced support for this method in React Native via commit [f01c4e2a14c194c7a02bc5afe1900573af02b0c7](https://github.com/facebook/react-native/commit/f01c4e2a14c194c7a02bc5afe1900573af02b0c7).
> But [it has been reversed](https://github.com/facebook/react-native/commit/eb40b09bfd0e617912663fb056d0bdea85ce0c2f) for performance concerns. Hopefully, one day this will be almost entirely automated.

##### 1. Copy and rename assets to the resource font folder

```sh
mkdir android/app/src/main/res/font
cp /tmp/raleway/*.ttf android/app/src/main/res/font
```

We must rename the font files following these rules:

- Replace `-` with `_`;
- Replace any uppercase letter with its lowercase counterpart.

You can use the below bash script (make sure you give the font folder as first argument):

```bash
#!/bin/bash
# fixfonts.sh

typeset folder="$1"
if [[ -d "$folder" && ! -z "$folder" ]]; then
  pushd "$folder";
  for file in *.ttf; do
    typeset normalized="${file//-/_}";
    normalized="${normalized,,}";
    mv "$file" "$normalized"
  done
  popd
fi
```

```sh
./fixfonts.sh /path/to/root/FontDemo/android/app/src/main/res/font
```

##### 2. Create the definition file

Create the `android/app/src/main/res/font/raleway.xml` file with the below content:

```xml
<?xml version="1.0" encoding="utf-8"?>
<font-family xmlns:app="http://schemas.android.com/apk/res-auto">
    <font app:fontStyle="normal" app:fontWeight="100" app:font="@font/raleway_thin" />
    <font app:fontStyle="italic" app:fontWeight="100" app:font="@font/raleway_thinitalic"/>
    <font app:fontStyle="normal" app:fontWeight="200" app:font="@font/raleway_extralight" />
    <font app:fontStyle="italic" app:fontWeight="200" app:font="@font/raleway_extralightitalic"/>
    <font app:fontStyle="normal" app:fontWeight="300" app:font="@font/raleway_light" />
    <font app:fontStyle="italic" app:fontWeight="300" app:font="@font/raleway_lightitalic"/>
    <font app:fontStyle="normal" app:fontWeight="400" app:font="@font/raleway_regular" />
    <font app:fontStyle="italic" app:fontWeight="400" app:font="@font/raleway_italic"/>
    <font app:fontStyle="normal" app:fontWeight="500" app:font="@font/raleway_medium" />
    <font app:fontStyle="italic" app:fontWeight="500" app:font="@font/raleway_mediumitalic"/>
    <font app:fontStyle="normal" app:fontWeight="600" app:font="@font/raleway_semibold" />
    <font app:fontStyle="italic" app:fontWeight="600" app:font="@font/raleway_semibolditalic"/>
    <font app:fontStyle="normal" app:fontWeight="700" app:font="@font/raleway_bold" />
    <font app:fontStyle="italic" app:fontWeight="700" app:font="@font/raleway_bolditalic"/>
    <font app:fontStyle="normal" app:fontWeight="800" app:font="@font/raleway_extrabold" />
    <font app:fontStyle="italic" app:fontWeight="800" app:font="@font/raleway_extrabolditalic"/>
    <font app:fontStyle="normal" app:fontWeight="900" app:font="@font/raleway_black" />
    <font app:fontStyle="italic" app:fontWeight="900" app:font="@font/raleway_blackitalic"/>
</font-family>
```

##### 3. Register the new font

In `android/app/src/main/java/**/MainApplication.java`, register the font inside `onCreate` method:

```diff
--- a/android/app/src/main/java/com/fontdemo/MainApplication.java
+++ b/android/app/src/main/java/com/fontdemo/MainApplication.java
@@ -7,6 +7,7 @@ import com.facebook.react.ReactApplication;
 import com.facebook.react.ReactInstanceManager;
 import com.facebook.react.ReactNativeHost;
 import com.facebook.react.ReactPackage;
+import com.facebook.react.views.text.ReactFontManager;
 import com.facebook.soloader.SoLoader;
 import java.lang.reflect.InvocationTargetException;
 import java.util.List;
@@ -43,6 +44,7 @@ public class MainApplication extends Application implements ReactApplication {
   @Override
   public void onCreate() {
     super.onCreate();
+    ReactFontManager.getInstance().addCustomFont(this, "Raleway", R.font.raleway);
     SoLoader.init(this, /* native exopackage */ false);
     initializeFlipper(this, getReactNativeHost().getReactInstanceManager());
   }

```

## iOS

On iOS, things will get much easier. We will basically just need to use React Native asset link functionnality.

### Copy font files to assets folder

```sh
mkdir -p assets/fonts
cp /tmp/raleway/*.ttf assets/fonts
```

### Add `react-native.config.js` 

```js
module.exports = {
  project: {
    ios: {},
    android: {},
  },
  assets: ['./assets/fonts'],
};
```

### Link

```sh
react-native link
```