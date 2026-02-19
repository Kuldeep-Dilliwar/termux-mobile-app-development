# termux-mobile-app-development
How to build android applications in termux? Here is how to: This repo contains guide to build android applications with using termux..

## Update everything first:
```
pkg update -y && pkg upgrade
```
## Installing termux dependencies:
```
pkg install aapt apksigner build-essential lld android-tools sqlite git tree which wget curl git openjdk-17 kotlin gradle binutils build-essential fzf aidl lld d8 aapt2
```
## To install android sdk version 34 (termux official bins only supports for sdk 34):
```
mkdir -p ~/android-sdk/licenses

echo "8933bad161af4178b1185d1a37fbf41ea5269c55" >  ~/android-sdk/licenses/android-sdk-license
echo "d56f5187479451eabf01fb78af6dfcb131a6481e" >> ~/android-sdk/licenses/android-sdk-license
echo "24333f8a63b6825ea9c5514f83c2829b004d1fee" >> ~/android-sdk/licenses/android-sdk-license
```
## Clone the repo:
```
git clone https://github.com/Kuldeep-Dilliwar/bare-minimum-android-app.git
cd bare-minimum-android-app
```
## (Skip) Or Use Gemini to write hello world android apk code:
```
yes | pkg update && yes | pkg i nodejs && clear && \
yes | GYP_DEFINES="android_ndk_path=$PREFIX" npm install -g @google/gemini-cli --ignore-scripts && clear && \
gemini
```
## Pointing to aarch64 bin for aapt2 & Pointing to android sdk folder:
```
echo "android.aapt2FromMavenOverride=/data/data/com.termux/files/usr/bin/aapt2"  > gradle.properties
echo "sdk.dir=/data/data/com.termux/files/home/android-sdk"                      > local.properties
```
## Triking gradle to install android sdk autometically. We will get error first time (gradle will see the licences and download the android sdk 34 autometically):
```
gradle assembleDebug
```

## Symlinking to installed aarch64 bins:

```
SDK_ROOT="$HOME/android-sdk"
BUILD_TOOLS="$SDK_ROOT/build-tools/34.0.0"
PLATFORM_TOOLS="$SDK_ROOT/platform-tools"

mobilize_tool() {
    local folder="$1"
    local tool="$2"
    
    if [ ! -f "$folder/$tool" ]; then
        echo "⚠️  Skipping $tool: File not found in SDK folder."
        return
    fi

    local termux_path=$(which "$tool")
    
    if [ -n "$termux_path" ] && [ -x "$termux_path" ]; then
        echo "✅ FOUND: $termux_path"
        echo "   -> Replacing $tool..."
        rm -f "$folder/$tool"
        ln -sf "$termux_path" "$folder/$tool"
    else
        echo "❌ MISSING: Termux does not have '$tool'. Keeping original file."
    fi
}

mobilize_tool "$BUILD_TOOLS" "aapt"
mobilize_tool "$BUILD_TOOLS" "aapt2"
mobilize_tool "$BUILD_TOOLS" "aidl"
mobilize_tool "$BUILD_TOOLS" "apksigner"
mobilize_tool "$BUILD_TOOLS" "d8"
mobilize_tool "$BUILD_TOOLS" "lld"
mobilize_tool "$BUILD_TOOLS" "aapt"

mobilize_tool "$PLATFORM_TOOLS" "adb"
mobilize_tool "$PLATFORM_TOOLS" "fastboot"
mobilize_tool "$PLATFORM_TOOLS" "sqlite3"
```
## (Skip) Checking the symlinks:
```
find ~/android-sdk \
    \( -type f -executable -o -name "*.so" -o -type l \) \
    -not -path "*/.*" \
    -exec ls -ldF --color=always {} +
```
## Build again
```
gradle assembleDebug
```
