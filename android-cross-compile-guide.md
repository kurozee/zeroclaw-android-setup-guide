# Android cross‑compile with cargo-android on WSL + Termux targets

This guide shows how to set up the Android SDK/NDK on WSL, configure environment variables, and build Rust binaries for Android (aarch64‑linux‑android) using `cargo-android`. It assumes a 64‑bit Android device (e.g. Samsung Note 20 Ultra, arm64‑v8a) and uses API 24 as the minimum Android version while still running on modern phones. [browser.geekbench](https://browser.geekbench.com/v5/cpu/baseline/10349832)

## 1. Prerequisites on WSL

On your WSL distro (Ubuntu/Debian‑like):

```sh
sudo apt update
sudo apt install -y curl unzip openjdk-17-jdk
```

Java is required for `sdkmanager`. [digitalocean](https://www.digitalocean.com/community/tutorials/how-to-install-java-with-apt-on-ubuntu-22-04)

## 2. Install Android command‑line tools (SDK skeleton)

From your WSL home:

```sh
cd ~
mkdir -p "$HOME/Android/cmdline-tools/latest"
curl -o cmdline-tools.zip https://dl.google.com/android/repository/commandlinetools-linux-11076708_latest.zip
unzip cmdline-tools.zip -d "$HOME/Android/cmdline-tools/latest"
rm cmdline-tools.zip
```

If the extracted archive nested a `cmdline-tools` folder inside `latest` (common case), normalize the layout:

```sh
cd "$HOME/Android/cmdline-tools/latest"
mv cmdline-tools/* .
rmdir cmdline-tools
```

After this, the tools live at:

```sh
$HOME/Android/cmdline-tools/latest/bin/sdkmanager
```



## 3. Set core environment variables (WSL)

In your current shell:

```sh
export ANDROID_SDK_ROOT="$HOME/Android"
export PATH="$ANDROID_SDK_ROOT/cmdline-tools/latest/bin:$ANDROID_SDK_ROOT/platform-tools:$PATH"
```

Confirm `sdkmanager` works:

```sh
sdkmanager --list
```

If you see a Java error, set `JAVA_HOME` (example path for OpenJDK 17):

```sh
JAVA_BIN="$(readlink -f "$(which java)")"
export JAVA_HOME="${JAVA_BIN%/bin/java}"
export PATH="$JAVA_HOME/bin:$PATH"
```

Then `sdkmanager --list` should succeed. [kontext](https://kontext.tech/project/javaprogramming/article/install-open-jdk-on-wsl)

Add these exports to `~/.bashrc` or `~/.zshrc` later for persistence.

## 4. Install SDK platform + NDK with sdkmanager

Install platform‑tools, an Android platform (API 34 for tooling), and NDK r26.1.10909125:

```sh
sdkmanager "platform-tools" "platforms;android-34" "ndk;26.1.10909125"
```

This downloads the NDK into:

```sh
ls "$ANDROID_SDK_ROOT/ndk"
# e.g. shows: 26.1.10909125
```

So your NDK path is:

```sh
$ANDROID_SDK_ROOT/ndk/26.1.10909125
```

Using a recent NDK (r26) is recommended and supports building back to API 21+, as long as you choose a suitable `ANDROID_API`. [developer.android](https://developer.android.com/ndk/downloads/revision_history)

## 5. Choose ANDROID_API (Termux / device check)

On your Android device in Termux, you can confirm ABI and API level:

```sh
echo "== Termux / Android build info =="

echo -n "ABI (uname -m): "
uname -m

echo -n "Primary ABI (ro.product.cpu.abi): "
getprop ro.product.cpu.abi

echo -n "Android API level (ro.build.version.sdk): "
API_LEVEL="$(getprop ro.build.version.sdk)"
echo "$API_LEVEL"
```

On a Samsung Note 20 Ultra you should see `aarch64` / `arm64-v8a` and an API level such as `33`. [browser.geekbench](https://browser.geekbench.com/v6/cpu/7868795)

For wide compatibility, pick `ANDROID_API=24` (Android 7.0) for your builds:

```sh
export ANDROID_API=24
```

Executables built for older Android versions run on newer versions, but not always the reverse, so choosing 24 gives you a safe minSdk while still running on your phone (API 33). [apilevels](https://apilevels.com)

## 6. Final environment exports (WSL)

Add the following to `~/.bashrc` (adjust NDK version if needed):

```sh
export ANDROID_SDK_ROOT="$HOME/Android"
export ANDROID_NDK_ROOT="$ANDROID_SDK_ROOT/ndk/26.1.10909125"
export ANDROID_API=24
export JAVA_HOME="/usr/lib/jvm/java-17-openjdk-amd64"  # adjust if different
export PATH="$JAVA_HOME/bin:$ANDROID_SDK_ROOT/cmdline-tools/latest/bin:$ANDROID_SDK_ROOT/platform-tools:$PATH"
```

Reload your shell:

```sh
source ~/.bashrc
```

Now `sdkmanager --list` and other tools should work without extra setup. [developer.android](https://developer.android.com/ndk/downloads)

## 7. Install Rust target and cargo-android

Install the `cargo-android` wrapper and the Android Rust target for 64‑bit ARM:

```sh
# From WSL:
cargo install --git https://github.com/chenxiaolong/cargo-android
rustup target add aarch64-linux-android
```

`cargo-android` is a bare‑bones wrapper that injects `ANDROID_NDK_ROOT` and `ANDROID_API` when the target is an Android one. [github](https://github.com/chenxiaolong/cargo-android/)

## 8. Build a Rust project for Android (aarch64)

From your Rust crate root (on WSL):

```sh
# Release build for Android aarch64:
cargo android build --target aarch64-linux-android --release
```

If `--target` is an Android target (e.g. `aarch64-linux-android`), `cargo-android` sets up the needed environment variables and calls `cargo` internally; otherwise, it falls back to a normal `cargo` invocation. [github](https://github.com/chenxiaolong/cargo-android/)

The resulting binary will be under:

```sh
target/aarch64-linux-android/release/<your-binary-name>
```

You can then package it or push it to your device (via `adb`, Termux, or inside an APK) as needed. [developer.android](https://developer.android.com/ndk/guides/other_build_systems)

## 9. Optional: Termux‑side quick check for planning

If you want a tiny helper snippet in Termux that both prints your ABI/API and proposes a sane `ANDROID_API` (<= your device, ≥ 24 where possible):

```sh
API_LEVEL="$(getprop ro.build.version.sdk)"

echo "== Termux / Android build info =="
echo -n "ABI (uname -m): "
uname -m
echo -n "Primary ABI (ro.product.cpu.abi): "
getprop ro.product.cpu.abi
echo "Android API level (ro.build.version.sdk): $API_LEVEL"

if [ -n "$API_LEVEL" ]; then
  if [ "$API_LEVEL" -ge 24 ]; then
    SUGGESTED_API=24
  else
    SUGGESTED_API="$API_LEVEL"
  fi
  echo
  echo "Suggested ANDROID_API for builds: $SUGGESTED_API"
  echo "export ANDROID_API=$SUGGESTED_API"
fi
```

Use that suggestion when configuring your WSL build environment so that the binaries you produce will run on your device and most reasonably recent Android versions. [api.suwish](http://api.suwish.com/android/ndk/guides/abis.html)

***
