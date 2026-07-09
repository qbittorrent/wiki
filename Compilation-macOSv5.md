# Building qBittorrent on macOS

This is a quick and dirty updated guide to building qBittorrent on macOS. 

## Table of Contents

- [Conventions Used](#conventions-used-in-this-document)
- [Build Instructions](#build-instructions)
- [Running qBittorrent](#running-qbittorrent)

---

## Conventions Used in This Document

The following directory structure will be used throughout this document:

```text
$HOME/GitHub/qBittorrent     <-- qBittorrent source directory
$HOME/qt/6.5.3/macos         <-- Qt installation directory
/opt/homebrew                <-- Homebrew installation (dependencies)
```

All commands should be issued from your home directory (`$HOME`) or the specific directory mentioned in each step.

---

## Build Instructions


### Step 1: Install Build Tools

```bash
# Install Homebrew (if not already installed)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install Xcode Command Line Tools
xcode-select --install

# Install build dependencies
brew install cmake pkg-config
```

### Step 2: Install Required Libraries

qBittorrent requires several libraries. Install them via Homebrew:

```bash
# Install OpenSSL 3.x
brew install openssl@3

# Install libtorrent-rasterbar (2.0.x series)
brew install libtorrent-rasterbar

# Install Boost 1.85 
brew install boost@1.85
```


### Step 3: Install Qt

Qt is not available via Homebrew in the exact version we need, so we'll use `aqt` (Another Qt Installer):

```bash
# Install aqt
pipx install aqtinstall

# Create Qt installation directory
mkdir -p ~/qt

# Install Qt 6.5.3 for macOS
aqt install-qt mac desktop 6.5.3 clang_64 --outputdir ~/qt
```

This downloads Qt 6.5.3 to `~/qt/6.5.3/macos/`. Installation takes a few minutes.

Verify Qt installation:

```bash
ls ~/qt/6.5.3/macos/lib/cmake/Qt6/Qt6Config.cmake
```

The file should exist.

### Step 4: Download qBittorrent Source Code

```bash
# Create source directory
mkdir -p ~/GitHub
cd ~/GitHub

# Clone qBittorrent repository
git clone https://github.com/qbittorrent/qBittorrent.git
cd qBittorrent
```

This clones the latest `master` branch. To then build a specific version:

```bash
# List available tags
git tag

# Checkout specific version (example)
git checkout release-5.1.4
```

### Step 5: Configure the Build

Set up environment variables to help CMake find dependencies:

```bash
# Set Qt path
export QT_ROOT="$HOME/qt/6.5.3/macos"
export Qt6_DIR="$QT_ROOT/lib/cmake/Qt6"

# Set Boost path
export BOOST_ROOT="/opt/homebrew/opt/boost@1.85"

# Prevent CMake from using wrong Boost version
export CMAKE_IGNORE_PATH="/opt/homebrew/Cellar/boost/1.90.0"
```

**Note**: The `CMAKE_IGNORE_PATH` prevents CMake from accidentally finding a newer Boost version if installed.

Now configure the build:

```bash
cmake -S . -B build \
    -DCMAKE_BUILD_TYPE=Release \
    -DQt6_DIR="$Qt6_DIR" \
    -DCMAKE_PREFIX_PATH="$QT_ROOT;$BOOST_ROOT" \
    -DBOOST_ROOT="$BOOST_ROOT" \
    -DCMAKE_IGNORE_PATH="$CMAKE_IGNORE_PATH"
```

**What this does**:
- `-S .` - Source directory (current directory)
- `-B build` - Build directory (will be created)
- `-DCMAKE_BUILD_TYPE=Release` - Build optimized release version
- `-DQt6_DIR` - Tells CMake where to find Qt
- `-DCMAKE_PREFIX_PATH` - Additional search paths for dependencies
- `-DBOOST_ROOT` - Explicitly set Boost location
- `-DCMAKE_IGNORE_PATH` - Paths to ignore (prevents finding wrong Boost)


### Step 6: Compile qBittorrent

```bash
cmake --build build
```

Or use multiple CPU cores to speed up compilation:

```bash
cmake --build build -j$(sysctl -n hw.ncpu)
```

**Build time**: 15-30 minutes depending on your system.

When complete, the application will be at:

```
build/qbittorrent.app
```

### Step 7: Bundle Qt Frameworks

The built application needs Qt frameworks bundled into it to run standalone:

```bash
~/qt/6.5.3/macos/bin/macdeployqt build/qbittorrent.app -verbose=2
```

This copies all required Qt frameworks into `qbittorrent.app/Contents/Frameworks/`.

### Step 8: Sign the Application

macOS requires applications to be signed to run properly - signing it this way is perfect for use for yourself, not for distribution.

```bash
codesign --force --deep --sign - build/qbittorrent.app
```

### Step 9: Remove Quarantine Attributes (if needed)

If you plan to copy the app elsewhere, remove quarantine attributes:

```bash
xattr -cr build/qbittorrent.app
```

---

## Running qBittorrent

For easier access, copy the app to your Applications folder:

```bash
cp -r ~/GitHub/qBittorrent/build/qbittorrent.app /Applications/
```


### First Launch

On first launch, macOS may display a security warning since the app is not from the App Store.

**If you see "Cannot be opened because the developer cannot be verified"**:

1. Right-click (or Control-click) on qbittorrent.app
2. Select "Open" from the context menu
3. Click "Open" in the dialog that appears

This only needs to be done once. After this, you'll be able launch the app normally.

### Creating a DMG for Distribution

To create a distributable DMG image:

```bash
~/qt/6.5.3/macos/bin/macdeployqt ~/GitHub/qBittorrent/build/qbittorrent.app -dmg
```

This creates `qbittorrent.dmg` in the build directory.

