# ANGLE for Android Readme

Additional ANGLE developer instructions for all platforms (Windows, Linux, MacOS, and Android) are
available [here](https://chromium.googlesource.com/angle/angle/+/master/doc/DevSetup.md).

# Download

The full ANGLE for Android APK build uses the Chromium and Android build systems, so both the
Chromium and Android repos must be cloned.   Note that these instructions are only focused on
building with Linux, since the Android build process is only supported on Linux and MacOS.

## Chromium

The following is a summary of the
[Checking out and building Chromium for Android](https://chromium.googlesource.com/chromium/src/+/master/docs/android_build_instructions.md)
page. Please refer to it for more details.

### Install depot_tools
Clone the depot_tools repository:
```bash
mkdir /path/to/depot_tools && cd /path/to/depot_tools
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
```

Add depot_tools to the end of your PATH (you will probably want to put this in your `~/.bashrc` or
`~/.zshrc`).
```bash
export PATH="$PATH:/path/to/depot_tools"
```

### Get the Chromium Code
Clone the Chromium repository:
```bash
mkdir /path/to/chromium && cd /path/to/chromium
fetch --nohooks android
```

## ANGLE
To use the ToT from the ANGLE repository in the Chromium build, modify Chromium's `.gclient` file:
```bash
vi /path/to/chromium/.gclient
```

Update the `custom_deps` section:

```
solutions = [
  {
    "url": "https://chromium.googlesource.com/chromium/src.git",
    "managed": False,
    "name": "src",
    "custom_deps":
    {
      "src/third_party/angle": None,
    },
    "custom_vars": {},
  },
]
target_os=["android"]
```

Bootstrap the ANGLE build so it's got the correct `.gclient`:

```
timvp@timvp:~/code/chromium/src/third_party/angle$ python scripts/bootstrap.py
```

Now update ANGLE to fetch the ToT.

## Android

The following is a summary of the directions available from the
[Downloading the Source](https://source.android.com/setup/build/downloading) page. Please refer to
it for more details.

### Install Repo

```bash
mkdir /path/to/bin/repo
PATH=/path/to/bin/repo:$PATH
curl https://storage.googleapis.com/git-repo-downloads/repo > /path/to/bin/repo
chmod a+x /path/to/bin/repo
```

### Initialize Repo Client

1. Create a working directory to hold the Android source code.

```bash
mkdir /path/to/android && cd /path/to/android
```

1. Configure git with your real name and email address. To use the Gerrit code-review tool, you will
need an email address that is connected with a registered Google account. Make sure this is a live
address at which you can receive messages. The name that you provide here will show up in
attributions for your code submissions.

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

1. Run repo init to bring down the latest version of Repo with all its most recent bug fixes. You
must specify a URL for the manifest, which specifies where the various repositories included in the
Android source will be placed within your working directory.

```bash
repo init -u https://android.googlesource.com/platform/manifest
```

To check out a branch other than "master", specify it with -b. For a list of branches, see
[Source Code Tags and Builds](https://source.android.com/setup/start/build-numbers.html#source-code-tags-and-builds).

```bash
repo init -u https://android.googlesource.com/platform/manifest -b android-4.0.1_r1
```

### Downloading the Android source tree

To pull down the Android source tree to your working directory from the repositories as specified in
the default manifest, run

```bash
repo sync
```

The Android source files will be located in your working directory under their project names. The
initial sync operation will take an hour or more to complete.

# Update Source Code
## Chromium
In a shell separate from the Android shell:
```bash
cd /path/to/chromium
git rebase-update
gclient sync
```

## ANGLE
In a shell separate from the Android shell:
```bash
cd /path/to/chromium/src/third_party/angle
git rebase-update
gclient sync
```

## Android
In a shell separate from the Chromium shell:
```bash
cd /path/to/android
repo sync -c -j40
repo rebase
```

# Build
## Build ANGLE Libraries
The first time you build the ANGLE for Android libraries on Linux:
```bash
cd /path/to/chromium/src/
./build/install-build-deps.sh
```

NOTE: If the `build/install-build-deps-android.sh` script fails, you may need to manually install
some packages yourself, using `sudo apt install`.  Then, re-run the command to ensure it succeeds.

The build arguments need to be configured first:

```bash
cd /path/to/chromium/src
gn args out/Default
```

NOTE: The `gn args` command will open an editor to enter the desired ANGLE build arugments.
The "Release" and "Debug" sections below contain values that can be used for either scenario.

Then the ANGLE shared objects can be built:

```bash
ninja -C out/Default third_party/angle:angle_apk
```

These commands will create the following ANGLE shared object libraries:
```bash
/path/to/chromium/src/out/Default/libEGL_angle.so
/path/to/chromium/src/out/Default/libfeature_support_angle.so
/path/to/chromium/src/out/Default/libGLESv1_CM_angle.so
/path/to/chromium/src/out/Default/libGLESv2_angle.so
```

### Release

```
target_os = "android"
target_cpu = "arm64"
is_debug = false
android32_ndk_api_level = 26
android64_ndk_api_level = 26
build_angle_deqp_tests = false
dcheck_always_on = true
ffmpeg_branding = "Chrome"
is_component_build = false
proprietary_codecs = true
symbol_level = 1
angle_enable_vulkan = true
angle_enable_vulkan_validation_layers = false
angle_libs_suffix = "_angle"
build_apk_secondary_abi = true
angle_enable_null = false
angle_force_thread_safety = true
```

### Debug

```
target_os = "android"
target_cpu = "arm64"
is_debug = true
android32_ndk_api_level = 26
android64_ndk_api_level = 26
build_angle_deqp_tests = true
dcheck_always_on = true
ffmpeg_branding = "Chrome"
is_component_build = false
proprietary_codecs = true
symbol_level = 2
angle_enable_vulkan = true
angle_enable_vulkan_validation_layers = true
angle_libs_suffix = "_angle"
build_apk_secondary_abi = true
angle_enable_null = false
angle_force_thread_safety = true
```

## Build ANGLE APK

Once the Chromium ANGLE build has completed, copy the generated 32b and 64b ANGLE shared object
libraries into the Android source tree:

```bash
cp /path/to/chromium/src/out/Default/android_clang_arm/lib*angle.so /path/to/android/vendor/unbundled_google/modules/ANGLEPrebuilt/lib/arm
cp /path/to/chromium/src/out/Default/lib*angle.so /path/to/android/vendor/unbundled_google/modules/ANGLEPrebuilt/lib/arm64
```

Build the ANGLE APK:

```bash
cd /path/to/android
source build/envsetup.sh
tapas GoogleANGLE <arm|arm64|x86|x86_64>
make -j
```

# Install

```bash
cd /path/to/android
adb install out/target/product/generic_arm64/system/product/priv-app/GoogleANGLE/GoogleANGLE.apk
```

# Using ANGLE

## Disable OpenGL ES Driver Preloading
Android Q disables OpenGL ES driver preloading by default.

If the device is running something other than Q:
```bash
adb root
adb shell setprop ro.zygote.disable_gl_preload 1
adb shell stop && adb shell start
```

## Skia
For best results, it's recommended to configure Skia to use the Vulkan back-end:
```bash
adb shell setprop debug.hwui.renderer skiavk
```

To revert back to the default Skia backend:
```bash
adb shell setprop debug.hwui.renderer none
```

## Enable ANGLE

## Developer Options

The ANGLE Developer Options allow for toggling various ANGLE settings:

Settings > Developer Options > ANGLE Preferences > Show dialog box when ANGLE is loaded

The "Select OpenGL Driver" section allows a user to specify which OpenGL ES driver is used for a
particular package.   Selecting an installed Application will present a dialog box with the following
options:
* default
    * Use the rules file to determine if ANGLE should be enabled or not.
* angle
    * Force enabling ANGLE.
* native
    * Force enabling the native driver.

Each (currently non-system) package can have a different value selected, which will persist across
reboots.

## `adb` Commands
`adb` can be used to set the necessary Global.Settings values to force package(s) to use ANGLE or
the native driver.

```bash
adb shell settings put global angle_gl_driver_selection_pkgs <package name>
adb shell settings put global angle_gl_driver_selection_values <driver>
```

The possible values for the `<driver>` value are: `angle`, `native`, and `default`, which correspond
to the Developer Options selections.

For example, to enable ANGLE for the dEQP package:
```bash
adb shell settings put global angle_gl_driver_selection_pkgs com.drawelements.deqp
adb shell settings put global angle_gl_driver_selection_values angle
```

Just like the Developer Options, this setting will persist across reboots.

### Enable ANGLE for All Packages
ANGLE can be enabled for all packages with the following command:
```bash
adb shell settings put global angle_gl_driver_all_angle 1
```

This setting can be disabled by setting it back to `0`:
```bash
adb shell settings put global angle_gl_driver_all_angle 0
```

Note that this setting is disabled in the Developer Options, because it currently makes the device
un-bootable.   The only way to enable/disable this setting is with `adb` to ensure that it's only
toggled when the user has access to `adb` so the device can be recovered if it's accidentally left
on across a reboot.

## Rules File method
Part of the Android platform driver loading process is to analyze a JSON "rules file" within the
ANGLE APK that specifies whether a particular package should use ANGLE or not based on certain
criteria (device, GPU, Vulkan driver version, etc.).

The current rules file can be found at:
```bash
/path/to/chromium/third_party/angle/src/feature_support_util/a4a_rules.json
```

Modifying this file will influence the choice made by the Android GLES loader, either to enable or
disable ANGLE for a package.

Additionally, a temporary rules file can be pushed to the device which will override the rules file
within the ANGLE APK.   After creating/modifying a new `a4a_rules.json` file, it can be used by
doing the following:

```bash
adb push a4a_rules.json /data/local/tmp/a4a_rules.json
adb shell setprop debug.angle.rules /data/local/tmp/a4a_rules.json
```

This temporary file will be removed from the device and the property will be reset during the boot
process, so these steps must be repeated after each boot completes.

# Verify ANGLE is Being Used

## Toast Message
A Toast message can be enabled with the "Show dialog box when ANGLE is loaded" setting in the ANGLE
Developer Options.   If enabled, whenever an app with ANGLE enabled is launched, a Toast message
containing the package name will be presented indicating ANGLE is enabled.

## Logcat
To verify that the application is using ANGLE with logcat with verbose logging enabled (a debug
Android build):
```bash
adb logcat
```

Verify the following text is output.   In this example, "com.drawelements.deqp" is opted into ANGLE.

```
02-27 13:01:11.914 12134 12134 V GraphicsEnvironment: Package 'com.drawelements.deqp' should use ANGLE = 'true'
[[[...]]]
02-27 13:01:11.947 12134 12157 I ANGLE   : Vulkan 1.1.87(Adreno (TM) 540 (0x05040001))
```

Note that the Vulkan driver vender/version will likely be different.

# System Properties

## `ro.gfx.angle.supported`

The system property `ro.gfx.angle.supported` indicates that ANGLE is in the currently running
Android image. This will cause CTS to verify that ANGLE can be enabled and disabled with the
Global.Settings and rules file.

This system property must be set to `true` in all Android images for devices that are required to
include ANGLE.

| Value   | Description
|---------|---------------------------------------------------------------
| `true`  | ANGLE is supported in the currently running Android image.
| `false` | ANGLE is NOT supported in the currently running Android image.

## `debug.angle.rules`

The system property `debug.angle.rules` can be set to the path to a temporary rules file that
overrides the default rules file present in the ANGLE APK. If the path is inaccessible, the default
rules file in the ANGLE APK will be used.

This property will be cleared during a reboot of the device.

| Value                     | Description
|---------------------------|------------------------------------------------------
| `/path/to/rulesFile.json` | The path to the location of the temporary rules file.

# Global Settings

## `angle_gl_driver_all_angle`

Force all Apps to use ANGLE. This overrides all other values that could disable ANGLE for a
particular App.

NOTE: This currently makes the device unbootable when enabled due to the Android boot process, how
ANGLE is loaded, and the capabilities of ANGLE. Due to this, the setting can only be
enabled/disabled with `adb` and not the Developer Options.

| Value | Description
|-------|------------------------------------
| `0`   | Do not force all Apps to use ANGLE.
| `1`   | Force all Apps to use ANGLE.

## `show_angle_in_use_dialog_box`

Show a dialog box (Toast message) when the App is launched that indicates it is using ANGLE.

| Value | Description
|-------|----------------------------
| `0`   | Do not show the dialog box.
| `1`   | Show the dialog box.

## `angle_gl_driver_selection_pkgs`

The list of Packages that have their OpenGL driver selection being forced to a particular value.
This list of packages corresponds 1:1 to the list `angle_gl_driver_selection_values`.

| Value            | Description
|------------------|---------------------------------------------------------
| `<package name>` | A package name or comma-separated list of package names.

## `angle_gl_driver_selection_values`

The list of Packages that have their OpenGL driver selection being forced to a particular value.
This list of packages corresponds 1:1 to the list `angle_gl_driver_selection_pkgs`.

| Value     | Description
|-----------|------------------------------------------------------
| `angle`   | Force using ANGLE for the corresponding Package name.
| `native`  | Force using the native driver for the corresponding Package name.
| `default` | Use the default driver determined by the rules file for the corresponding Package name.

## `angle_whitelist`

The list of package names present in the default rules file included in the ANGLE APK. This
whitelist of package names is used to improve App startup time by only parsing the default rules
file if the package name is present in the rules file.

| Value            | Description
|------------------|---------------------------------------------------------
| `<package name>` | A package name or comma-separated list of package names.
