## 3.0.0-alpha.1
- Add `enableRenderCache` parameter.

```dart
Lottie.asset('assets/complex_animation.json',
  enableRenderCache: true,
)
```

It allows to opt into a mode where the frames of the animation are rendered lazily in an offscreen cache.
Subsequent runs of the animation will be very cheap to render.

This is useful is the animation is complex and can consume a lot of energy from the battery.
It's a trade-off to lower the CPU usage at the cost of an increased memory usage.

The render cache is managed internally and will release the memory once the animation is disposed. 
The cache is shared between all animations. If 2 `Lottie` widget are rendered at the same size, they will render only 
once.

Any change in the configuration of the animation (delegates, frame rate etc...) will clear the cache.
Any change in the size will invalidate the cache. The cache use the final size visible on the screen (with all 
transforms applied).

- Allow to load Telegram Stickers (.tgs)

```dart
Lottie.asset(
  'sticker.tgs',
  decoder: LottieComposition.decodeGZip,
)
```

- Expose a hook to customize how to decode zip archives. This is useful for dotlottie archives (.lottie) when we want
to specify a specific .json file inside the archive

```dart
Lottie.asset(
  'animation.lottie',
  decoder: customDecoder,
);

Future<LottieComposition?> customDecoder(List<int> bytes) {
  return LottieComposition.decodeZip(bytes, filePicker: (files) {
    return files.firstWhere((f) => f.name == 'animations/cat.json');
  });
}
```

- Remove the name property from `LottieComposition`
- `imageProviderFactory` is not used in .zip file by default anymore.
To restore the old behaviour, use:
```dart
Future<LottieComposition?> decoder(List<int> bytes) {
  return LottieComposition.decodeZip(bytes, imageProviderFactory: imageProviderFactory);
}

Lottie.asset('anim.json', imageProviderFactory: imageProviderFactory, decoder: decoder)
```
- Disable gradient cache optimization when `ValueDelegate.gradientColor` is used
- Use `DefaultAssetBundle.of` in `AssetLottie` before fallback to `rootBundle`
- Add `BuildContext` optional parameter in `LottieProvider.load`
- Fixed varying opacity stops across keyframes in the same gradient
- Fixed rounded corners for non-closed curves

## 2.7.0
- Support loading Fonts from a zip file
- Fix a bug in Text with strokes
- Fix gradient interpolation for opacity stops beyond the last color stop
- Add color value callback to solid layer

## 2.6.0
- Accept `List<int>` instead of `Uint8List` in `LottieComposition.fromBytes`
- Stroke line cap defaults to butt instead of square

## 2.5.0
- Add layer-level opacity option to LottieOptions
- Fix TextLayer opacity calculation

## 2.4.0
- Require minimum Dart 3.0.0 and Flutter 3.10.0
- Fix a parsing bug when the name property in RoundedCorner was null

## 2.3.2
- Fix a bug when running on the web due to [bitwise operations difference](https://dart.dev/guides/language/numbers#bitwise-operations).

## 2.3.1
- Fix an assertion for null `ShapeTrimPathType.type`.

## 2.3.0
- Fixed a failed assertion (`dirty: is not true`) when calling `setState` inside `onLoaded` callback.
- Update point text y offset
- Makes static compositions not animate by default

## 2.2.0
Apply the latest fixes from Lottie-Android:

- Overhaul text layout
- Fix rounded corners modifying already rounded corners
- Support box position in Document Data
- Allow interpolating between gradients with different opacity stops
- Add support for gradient opacity stops

## 2.1.0
- Improve the cache to ensure that there is not an empty frame each time we load an animation.  
  The method `AssetLottie('anim.json').load()` returns a `SynchronousFuture` if it has been loaded previously.
- Expose the `LottieCache` singleton.
  It allows to change the cache behaviour and clear the entries.

```dart
void main() {
  Lottie.cache.maximumSize = 10;
  Lottie.cache.clear();
  Lottie.cache.evict(NetworkLottie('https://lottie.com/anim.json'));
}
```

## 2.0.0
- **Breaking change**: the lottie widget will be smaller if it relies on the intrinsic size of the composition. 

Previously the lottie parser was automatically multiplying the size of the composition by `window.devicePixelRatio`.
This was incorrect as it results in a widget of a different size depending on the pixel ratio of the monitor. 
Furthermore, it created some bugs when the property `window.devicePixelRatio` was not available immediately at the start
of the app (on Android release builds). 

The code can be adapted to specify explicitly the size of the animation with `width`, `height` and `fit` properties.

```dart
Scaffold(
  body: Center(
    child: Lottie.asset(
      'assets/LottieLogo1.json',
      height: 800,
      fit: BoxFit.contain,
    ),
  ),
);
```

## 1.4.3
- Fixed some lints with Flutter 3.3. 

## 1.4.2
- Use `FilterQuality.low` as default to draw image layers.

## 1.4.1
- Allow `AlignmentGeometry` for `alignment`.

## 1.4.0
- Added `filterQuality` property to control the performance vs quality trade-off to use when drawing images

## 1.3.0
- Added support for rounded corners on shapes and rects
- Add support for text in dynamic properties (`ValueDelegate`)

Example:

```dart
Lottie.asset(
  'assets/DynamicText.json',
  delegates: LottieDelegates(values: [
    ValueDelegate.text(
      ['Text layer'], // The path to the text element to change
      value: 'The new text',
    ),
  ]),
)
```

- Improve stroke with offset
- Add support for reversed polystar paths
- Enforce order of operations to avoid rounding errors

## 1.2.2
- Internal maintenance: fix lints for Flutter 2.10

## 1.2.1
- Fix: Revert Cubic to `PathInterpolator.cubic`

## 1.2.0
- Add support for gaussian blurs.

Example to blur some elements dynamically:
  
```dart
Lottie.asset(
  'assets/AndroidWave.json',
  delegates: LottieDelegates(values: [
    ValueDelegate.blurRadius(
      ['**'], // The path to the element to blur
      value: 20,
    ),
  ]),
)
```

- Add support for drop shadows.

Example to add a shadow dynamically:
```dart
Lottie.asset(
  'assets/animation.json',
  delegates: LottieDelegates(values: [
    ValueDelegate.dropShadow(
      ['**'], // The path to the elements with shadow
      value: const DropShadow(
        color: Colors.blue,
        direction: 140,
        distance: 60,
        radius: 10,
      ),
    ),
  ]),
)
```

## 1.1.0
- Add `errorBuilder` callback to provide an alternative widget in case an error occurs during loading.
```dart
Lottie.network(
  'https://example.does.not.exist/lottie.json',
  errorBuilder: (context, exception, stackTrace) {
    return const Text('😢');
  },
);
```

- Add `onWarning` to be notified when a warning occurs during the animation parsing or painting.
  Previously the warnings where written in an internal `logger`.
  ```dart
  Lottie.asset('animation.json'
    onWarning: (warning) {
      _logger.info(warning);
    },
  );
  ```
- Various bug fixes

## 1.0.1
- Implement `RenderBox.computeDryLayout`

## 1.0.0
- Migrate to null safety
- Fix some rendering bugs
- Add an image delegate to dynamically change images
- Allow to use an imageProviderFactory with a zip file

## 0.7.1
- Fix a crash for some lottie file with empty paths.

## 0.7.0+1
- Fix Flutter Web compilation error

## 0.7.0
- Performance improvement for complex animations.

## 0.6.0
- Runs the animation at the frame rate specified in the json file (ie. An animation encoded with a 20 FPS will only
  be paint 20 times per seconds even though the AnimationController will invalidate the widget 60 times per seconds).  
  A new property `frameRate` allows to opt-out this behavior and have the widget to repaint at the device frame rate 
   (`FrameRate.max`).
- Automatically add a `RepaintBoundary` around the widget. Since `Lottie` animations are generally complex to paint, a
   `RepaintBoundary` will separate the animation with the rest of the app and improve performance. A new property `addRepaintBoundary`
   allows to opt-out this behavior.
- Fix a bug where we would call `markNeedPaint` when the animation was not changing. This removes unnecessary paints in
  animations with static periods.

## 0.5.1
- Remove direct dependencies on dart:io to support Flutter Web

## 0.5.0
- Support loading animation from network in a web app
- Fix a couple of bugs with the web dev compiler

## 0.4.1
- Support color value stored as RGB, not RGBA 

## 0.4.0+1
- Support latest version of the `characters` package

## 0.4.0
- Disable "Merge paths" by default and provide an option to enable them.  
This is the same behavior as in Lottie-android.  
Merge paths currently don't work if the the operand shape is entirely contained within the
first shape. If you need to cut out one shape from another shape, use an even-odd fill type
instead of using merge paths.

Merge paths can be enabled with:
```dart
Lottie.asset('file.json', options: LottieOptions(enableMergePaths: true));
```


## 0.3.6
- Export the `Marker` class

## 0.3.5
- Fix a bug with a wrongly clipped rectangle. 

## 0.3.4
- Fix a bug with dashed path

## 0.3.3
- Fix a bug with rounded rectangle shape

## 0.3.2
- Fix a bug with "repeater" content

## 0.3.1
- Support dashed path

## 0.3.0+1
- Specify a version range for the dependency on `characters`.

## 0.3.0
- Add `LottieDelegates` a group of options to customize the lottie animation at runtime.
  ie: Dynamically modify color, position, size, text... of every elements of the animation.
- Correctly display Linear and Radial Gradients
- Integrate latest changes from Lottie-android

## 0.2.2
- Add a [repeat] parameter to specify if the automatic animation should loop.
- Add the [animate], [reverse], [repeat] properties on `LottieBuilder`
- Fix bug with `onLoaded` callback when the `LottieProvider` is changed

## 0.2.1
- Fix a big bug in the path transformation code. A lot more animations look correct now.

## 0.2.0+1
- Improve readme
- (internal) Add golden tests

## 0.2.0
- Support loading the animation and its images from a zip file
- Breaking: `LottieComposition.fromBytes` and `fromByteData` are now asynchronous.

## 0.1.4
- Support images in animation
- Basic support for text in animation (work in progress)

## 0.1.3
- Support Polystar shape
- Reorganize examples.

## 0.1.2
- Implement `Lottie.network`, `Lottie.file` and `Lottie.memory`

## 0.1.1
- Fix analysis lints

## 0.1.0
- Initial conversion of [lottie-android](https://github.com/airbnb/lottie-android) to Dart/Flutter
