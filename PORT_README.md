# imgui-java 1.90.0 + Android support (ported from alexytomi/imgui-java)

This package contains the **minimal set of patched files** that add Android NDK build support
to official imgui-java v1.90.0 (SpaiR), based on the changes in https://github.com/alexytomi/imgui-java
(commit ff785a0 + related).

## Files included

- `buildSrc/scripts/build.sh` – added `android` case
- `buildSrc/scripts/vendor_freetype.sh` – added Android FreeType cross-compile for 4 ABIs + mkdir fix
- `buildSrc/src/main/groovy/tool/generator/GenerateLibs.groovy` – Android BuildTarget, FreeType per-ABI paths, skip pack-natives for Android
- `imgui-binding/src/main/native/jni_jvm.cpp` – JNI_VERSION_1_6 for broader Android compatibility

## How to apply

1. Start from the official imgui-java-1.90.0 source tree (the zip you provided or a fresh checkout of tag v1.90.0).
2. Replace the four files above with the ones from this package (keep the exact directory structure).
3. Initialize submodules if needed:
   ```bash
   git submodule update --init --recursive
   ```
4. Build Android natives (requires Android NDK r21+ with the llvm prebuilt toolchain):

   ```bash
   export ANDROID_NDK_HOME=/path/to/android-ndk   # or NDK_HOME
   ./buildSrc/scripts/build.sh android
   ```

   The script will:
   - Cross-compile FreeType for arm64-v8a / armeabi-v7a / x86 / x86_64
   - Generate and run the jnigen Android Ant script (`build-android32.xml`)
   - Place the resulting shared libraries under `/tmp/imgui/dst/<abi>/libimgui-java.so`

## Notes

- FreeType is statically linked and enabled by default (same as upstream 1.90.0).
- On Android the library is named `libimgui-java.so` (no "64" suffix).
- The four ABIs are produced in one pass; you can package them into an AAR the same way the Amethyst / alexytomi projects do.
- Some extensions (ImGuiColorTextEdit, ImGuiFileDialog) remain commented out in the INCLUDES list of GenerateLibs.groovy; uncomment them if you need them on Android.
- The original alexytomi warning about "do not update past 1.86" is no longer applicable once these patches are applied.

## Verification

After a successful `build.sh android` you should see:

```
/tmp/imgui/dst/arm64-v8a/libimgui-java.so
/tmp/imgui/dst/armeabi-v7a/libimgui-java.so
/tmp/imgui/dst/x86/libimgui-java.so
/tmp/imgui/dst/x86_64/libimgui-java.so
```

## CI Workflow

The package also includes an updated GitHub Actions workflow:

`.github/workflows/ci.yml`

Key changes vs upstream 1.90.0:
- Matrix includes `android` (ubuntu-latest)
- Exports `NDK_HOME=$ANDROID_NDK_HOME` before calling `buildSrc/scripts/build.sh`
- Uses modern action versions (checkout@v6, setup-java@v5, upload-artifact@v7, etc.) matching the workflow you provided
- Windows/macOS matrix entries are commented out (uncomment if you want them)

Copy it to `.github/workflows/ci.yml` in your repo (or merge the android matrix + NDK export into your existing workflow).
