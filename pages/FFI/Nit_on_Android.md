Since `mnit` and its Android variant, `mnit_android`, has been added to the language library, Nit support compiling to the Android platform.

# So you want a software product line?

A software product line is a set of applications sharing a commom subset of features (or code when in development). We rely on this idea to create cross-platform applications in Nit with `mnit`. In the case where you target a specific platform (forever), you don't have to bother with it, write a single application and skip this section.

We begin by illustrating an usage of software product line using the  example of the Dino project. It is found in the Nit repository under examples/mnit_dino/.

The Dino project has 5 modules shared across all platforms:

* The higher level modules is `game_logic`, it is completely independent of any platform, graphic framework and application framework. It can be used to test the full logic using command line applications.

* `graphism`, `splash` and `fancy_dino` use class refinement to introduce new services based on the graphic framework of `mnit`. Separating business logic from the GUI is a good practice in any software. Class refinement just makes it easier and cleaner.

* `dino` describes the application logic by implementing an `App` from the `mnit` application framework. With its importations, it unites all of the code shared between platforms. By itself, it cannot be compiled to a working application.

As products, the Dino project defines 2 main modules for the actual working applications:

* `dino_linux` is the Linux version of the game. Besides the shared code, it imports only `mnit_linux`.

* `dino_android` is the Android version of the game. It imports `mnit_android` and defines behaviors specific to the Android platform. Most notably, the vibration from `mnit_android::vibration`.

All these 7 modules implement 2 products/applications. Plus, you could test the game logic using only `game_logic`.

`mnit` rely on a similar structure for any cross-platform application.

# Compiling the examples

Begin by installing the Android SDK and NDK. Update the environment variable PATH to include the path to the NDK and `path_to_android_sdk/tools`.

To compile `Dino`, run `make android` in the root of the package. It will generate an APK in the `bin/` directory.

Now take a look at the `Makefile`, the android rule is pretty straight forward. The Nit compiler detects the target plaform from the imported modules. Including the `mnit_android` modules also import `android`. The last one triggers the compilation to an APK instead of an executable.

# Libraries compatible with Android

## Classic Nit libraries

We'll divide the Nit libraries in 3 categories:
 
* All Nit libraries implemented in pure Nit, can be used on Android. This includes most of the standard library, `json`, `serialization`, `a_star`, `pipeline`, `poset`, `template` and `trees`.

* Libraries wrapping external C libraries will work on Android only if they are available in the NDK. As of now, the Nit libraries `glesv2`, `egl` and `jvm` are available. To see what extra libraries are available on Android and could be wrapped, take a look at the available C header in `path_to_your_ndk/platforms/android-{version}/arch-arm/usr/lib/`.

* Other libraries wrapping C libraries that are not available in the Android NDK will not work on Android. This includes `curl`, `gtk`, `curses`, etc. If needed, some of them can be ported to Android. As of now, only libpng is included in the generated projects. It is only used by `mnit_android` and is not directly available as a module.

## Android specific services in Nit

The Nit library offers a few (but growing) number of libraries for the Android platform.

Many services are included in the `mnit_android` module and are automatically made available to you:

* Redefines `print` to output to Android's logs.
* Load package assets using `App::load_asset`.
* Access sensor data.

TODO add vibration when integrated

# Frequently Asked Questions and Common Problems

### On installing the application on some devices I get "Package file was not signed correctly"

Usually this error is caused by an incorrect version of jarsigner used by the developer. You must use the one from JDK 6, not 7.
