## Note

This guide is a concise installation instruction based on the old guides [Compilation macOS](Compilation-macOS.md) and [Compilation macOS (x86_64, arm64, cross compilation).md](Compilation-macOS-(x86_64,-arm64,-cross-compilation).md)
The old guides can no longer compile due to recent breaking changes, but they still provide a lot more explanations for each build steps.

This guide has been tested for compiling on macOS Tahoe (26.0) with an M5 mac on 2026-03-22.
Compiling on other OS versions or architecture is likely as easy as changing the variable in the [Setup](#Setup) section.
This guide does not cover cross compilation for simplicity.

## Setup

1. create an empty folder to store all build related files
2. cd into it
3. Set the shell variables that will be used in later steps (assuming you use the same shell for the entire building process)
```bash
# note: 26.0 is Tahoe
# change those two fields as needed
OSX_TARGET=26.0
OSX_ARCHITECTURES=arm64

BUILD_PATH=`pwd`
# create sub folders
mkdir src ext
```

## boost

1. check the current dependency of boost in https://github.com/qbittorrent/qBittorrent/blob/master/INSTALL
2. download boost from https://www.boost.org/releases/latest/
3. unzip and copy the folder as `$BUILD_PATH/src/boost`
note: installation is not needed as only the headers are used

## openssl

1. check the current dependency of openssl in https://github.com/qbittorrent/qBittorrent/blob/master/INSTALL
2. download openssl from https://openssl-library.org/source
3. unzip and copy the folder as `$BUILD_PATH/src/openssl`
```bash
cd "$BUILD_PATH/src"
mkdir build-openssl && cd build-openssl
# note: `--api=1.1.1` is needed as libtorrent requires it as of 2.0.12
../openssl/config no-comp --api=1.1.1 no-deprecated no-dynamic-engine no-tests no-shared no-zlib --openssldir=/etc/ssl --prefix="$BUILD_PATH/ext" -mmacosx-version-min=10.14
make -j$(sysctl -n hw.ncpu)
make install_sw
```

## libtorrent

1. check the current dependency of libtorrent in https://github.com/qbittorrent/qBittorrent/blob/master/INSTALL
2. download libtorrent from https://github.com/arvidn/libtorrent/releases
3. unzip and copy the folder as `$BUILD_PATH/src/libtorrent`
```bash
cd "$BUILD_PATH/src"
cmake -S libtorrent -B build-libtorrent -D CMAKE_VERBOSE_MAKEFILE=ON -D CMAKE_PREFIX_PATH="$BUILD_PATH/src/boost;$BUILD_PATH/ext" -D CMAKE_CXX_STANDARD=17 -D CMAKE_CXX_EXTENSIONS=OFF -D CMAKE_OSX_DEPLOYMENT_TARGET=$OSX_TARGET -D CMAKE_OSX_ARCHITECTURES=$OSX_ARCHITECTURES -D CMAKE_BUILD_TYPE=Release -D BUILD_SHARED_LIBS=OFF -D deprecated-functions=OFF -D CMAKE_INSTALL_PREFIX=$BUILD_PATH/ext
cmake --build build-libtorrent -j$(sysctl -n hw.ncpu)
cmake --install build-libtorrent
```

## QT

```bash
cd "$BUILD_PATH/src"
git clone https://code.qt.io/qt/qt5.git
cd qt5
# note: latest version at time of writing, a recent version is needed for Tahoe support
git checkout v6.9.3
perl init-repository --module-subset=qtbase,qtsvg,qttools,qttranslations
cd "$BUILD_PATH/src"
mkdir build-qt && cd build-qt
# updated for v6.9.3
../qt5/configure -prefix "$BUILD_PATH/ext" -opensource -confirm-license -release -static -appstore-compliant -c++std c++17 -no-pch -I "$BUILD_PATH/ext/include" -L "$BUILD_PATH/ext/lib" -make libs -no-dbus -no-icu -qt-pcre -system-zlib -ssl -openssl-linked -no-cups -qt-libpng -qt-libjpeg -no-feature-testlib -platform macx-clang
make -j$(sysctl -n hw.ncpu)
make install
```

## qBittorrent

```bash
cd "$BUILD_PATH/src"
git clone https://github.com/qbittorrent/qBittorrent.git
cd qBittorrent
mkdir build && cd build
cmake -DCMAKE_PREFIX_PATH="$BUILD_PATH/src/boost;$BUILD_PATH/ext" -DCMAKE_CXX_STANDARD=17 -DCMAKE_CXX_EXTENSIONS=OFF -DCMAKE_OSX_DEPLOYMENT_TARGET=$OSX_TARGET -DCMAKE_BUILD_TYPE=Release ..
make -j$(sysctl -n hw.ncpu)
open qbittorrent.app
```
