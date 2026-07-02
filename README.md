# OnePlus IR Remote — Patched APK for Custom ROMs

A patched version of the OnePlus Consumer IR app (`com.oplus.consumerIRApp`) for use on custom ROMs based on [android_device_oneplus_ziti](https://github.com/pjgowtham/android_device_oneplus_ziti) that ship without certain proprietary OnePlus framework libraries.

---

## Background

The stock OnePlus IR Remote app depends on several proprietary OnePlus system framework APIs bundled in `oplus-fwk.jar`. Custom ROM builds often omit or update these libraries, causing the app to crash on launch or during use. This patched APK stubs out all missing framework calls so the app runs correctly without them.

## What Was Fixed

The following missing framework APIs were patched out and replaced with safe stubs:

| Missing API | Location | Fix |
|---|---|---|
| `OplusBuild$VERSION.SDK_VERSION` field | `i2/b.smali` | Hardcoded to OS version `0x23` |
| `OplusBuild$VERSION.SDK_SUB_VERSION` field | `i2/b.smali` | Hardcoded to `0x0` |
| `OplusFeatureConfigManager.getInstance()` | Multiple files | Stubbed to return `false` (except IR hardware check which returns `true`) |
| `DynamicFrameRateManager.getDynamicFrameRateType()` | `g1/d.smali` | Stubbed to return `0` |
| `DynamicFrameRateManager.setFrameRate()` | `g1/d.smali` | Stubbed to no-op |
| `OplusView` constructor + `setOverrideLightSourceGeometry()` | `h2/g.smali` | Removed entirely (shadow effect disabled) |
| `OplusOutline` constructor + `setSmoothRoundRect()` | Multiple COUI UI files | Redirected to standard `Outline.setRoundRect()` fallback |

### Files Modified

- `smali/i2/b.smali`
- `smali/h2/g.smali`
- `smali/g1/d.smali`
- `smali/j2/a.smali`
- `smali/h7/b.smali`
- `smali/h7/e0.smali`
- `smali/n0/d.smali`
- `smali/com/coui/appcompat/dialog/widget/COUIAlertDialogClipCornerLinearLayout$a.smali`
- `smali/com/coui/appcompat/poplist/RoundFrameLayout$a.smali`
- `smali/com/coui/appcompat/snackbar/COUINotificationSnackBar$a.smali`
- `smali/com/coui/appcompat/snackbar/COUISnackBar$d.smali`
- `smali_classes2/WifiManagerNative.smali`
- `smali_classes2/UserManagerNative.smali`

## Known Limitations

- **Three dot menu (⋮) does nothing** — the overflow menu relies on COUI UI components that depend on missing framework APIs. Core IR functionality is unaffected.
- **Shadow effects disabled** — `OplusView` shadow geometry is not applied. Purely cosmetic.
- **Smooth rounded corners** — COUI components fall back to standard Android rounded corners instead of OnePlus's squircle variant. Visually very similar.
- **Renaming Added Appliances** — Since the three dot menu doesnt work the added appliances cant be renamed once added.
## Functionality

Everything that matters works:

-  App launches
-  Add new IR device
-  Browse device brands and models
-  IR signal transmission (tested with AC)
-  Device control panel

## Requirements

- OnePlus device with IR blaster
- Custom ROM
- Android 14 or later

## Installation

1. Uninstall the existing IR Remote app if present:
   ```
   adb uninstall com.oplus.consumerIRApp
   ```
   Or uninstall via Settings → Apps on your device.

2. Install the patched APK:
   ```
   adb install IR-Remote-patched.apk
   ```
   Or sideload via a file manager.

> **Note:** Because this APK is signed with a different key than the stock OnePlus app, you must fully uninstall the original before installing this one.

## How It Was Patched

The APK was decompiled using [apktool](https://apktool.org), the missing framework calls identified via `adb logcat` crash analysis, stubbed out at the smali bytecode level, then rebuilt and signed with a debug keystore.

If a future ROM update breaks the app again, follow the same process:
1. `apktool d IR-Remote.apk -o ir`
2. Capture `adb logcat` crash output
3. Find the offending smali file via `Get-ChildItem -Recurse -Filter "*.smali" | Select-String "MissingClass"`
4. Replace `new-instance` / `invoke-static` / `sget` calls on missing classes with constants or no-ops
5. `apktool b ir -f -o patched.apk`
