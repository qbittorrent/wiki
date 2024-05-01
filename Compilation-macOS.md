Following this guide, anyone even completely unexperienced in C/C++ software building process can successfully build qBittorent. Only very basic skills in command line usage are required.

This guide could seem too detailed or even odd, but that was done intentionally, because first of all it aims to help novice users. Any experienced users may just overview it to find section they are interested in. For example, [the last one](#building-with-qmake) may be interesting for new contributors who want to debug or just poke around the code.

This guide describes build process using qBittorrent sources from `master` branch, but it also can be suitable for 4.3.x releases. It assumes only using of libtorrent 1.2.x series. Building qBittorrent 4.1.x series or 4.2.x with libtorrent 1.1.x is out of scope, similar concept can be used in any case, but various build options must be adjusted.

This guide assumes building **everything** (i.e. all dependencies) from sources, so whole build process is pretty time consuming (takes ~1 hour on old MacBook Pro Retina Mid 2014).

## Conventions used in this document

The following directory tree structure will be used through the document:

```text
$HOME/tmp/qbt
 |_ src         <-- directory for all sources
 |_ ext         <-- directory for all compiled dependencies
```

It is assumed that all listed commands should be issued from `$HOME/tmp/qbt/src` unless otherwise specified.

## Build environment setup

- XCode: default IDE and toolchain for macOS (includes the SDK and essential tools for native development)

  - Download and install XCode from the official [Apple App Store][xcode-appstore].
  - Launch XCode for the first time. You will be asked to install additional command-line tools, which you should do, as they are required for building Qt-based apps such as qBittorrent.

- CMake: a build-system generator, nowadays the de-facto standard for building C/C++ projects.

  - Download and install CMake from the [official website][cmake-off-site]. A GUI and some command-line tools will be installed. The GUI will not be needed for this tutorial.

Please note that XCode takes up several GiBs of space, and may take several minutes to install on your system.

## Required sources

qBittorrent has few required dependencies, and they also have their own. The full list of required sources (including qBittorrent itself) is next:

- OpenSSL 1.1.x
- Qt 5.12.x or above is required
- Boost 1.60 or above, 1.70 or above is recommended
- libtorrent (libtorrent-rasterbar) 1.2.x, 1.2.12 or above is required
- qBittorrent 4.3.x or above

Actually, some additional libraries (like zlib) are required, but they are available on each system and have stable binary interfaces, so there is no reason to build them from sources. Moreover, such libraries are part of the SDK.

## Downloading sources

Download and build instructions was split due to some macOS specific stuff, see "[Building sources](#building-sources)" section for details.

### Downloading OpenSSL

OpenSSL is available for download from [official website][openssl-site], just download archive for 1.1.x series. At the moment of writing it was 1.1.1g.

Place downloaded archive to `$HOME/tmp/qbt/src` and extract it here:

```sh
tar xf openssl-1.1.1g.tar.gz
```

### Downloading Qt

`git` is used for Qt download. It is much effective rather than using source archive, and brings a lot of advantages for build process (less configure options are required). Moreover, it requires less space and time rather than using sources from archive, due to only few required submodules can be downloaded, not all available/provided by default.

Clone official Qt5 super module repository:

```sh
git clone https://code.qt.io/qt/qt5.git
```

It is recommended to use stable releases, but using any other branch with newer Qt version is also possible. At the moment of writing Qt 5.14.2 was the latest, so use it. Switch (checkout) to desired branch/tag:

```sh
cd qt5
git checkout v5.14.2
```

Now top-level repo is initialized, but NO any Qt sources were downloaded. To download sources issue next command:

```sh
perl init-repository --module-subset=qtbase,qtmacextras,qtsvg,qttools,qttranslations
```

This downloads only required parts of Qt. Download process takes few minutes depending on connection speed. About ~500 MB will be downloaded.

### Downloading Boost

Boost is available for download from [official website][boost-site]. Latest available release is a good choice in most cases, so pick it firstly. If libtorrent build fails, download previous version. Preferable archive format is `.tar.bz2` or `.tar.gz`. At the moment of writing latest version was 1.73.0.

Place downloaded archive to `$HOME/tmp/qbt/src` (see [conventions](#conventions-used-in-this-document)) and extract it here:

```sh
tar xf boost_1_73_0.tar.bz2
```

### Downloading libtorrent

libtorrent RC_1_2 branch is used in this guide, but any 1.2.x release is also suitable. 1.2.12 and above is required.

Just clone [official repo][libtorrent-repo], RC_1_2 branch was default at the time of writing, so no `git checkout` is required:

```sh
git clone --recurse-submodules https://github.com/arvidn/libtorrent.git
```

It is also possible to download the release archive or git snapshot instead of cloning the repo.

### Downloading qBittorrent

qBittorrent master branch is used in this guide, but any 4.3.x release or v4_3_x branch can be used.

master branch is the default, so no additional `git checkout` is required. Just clone [official repo][qbittorrent-repo]:

```sh
git clone https://github.com/qbittorrent/qBittorrent.git
```

It is also possible to download the release archive or git snapshot instead of cloning the repo.

## Building sources

It is important to build all libraries with the same value of minimum supported macOS version, otherwise the build may fail at the final linker stage. Qt sets such value, so use it for any other libraries.

To find it, go to the Qt sources directory, open `qtbase/mkspecs/common/macx.conf` file and find `QMAKE_MACOSX_DEPLOYMENT_TARGET` variable in it.

This can be done through the command line using `grep` utility:

```sh
cd qt5
grep QMAKE_MACOSX_DEPLOYMENT_TARGET qtbase/mkspecs/common/macx.conf | grep -Eo -e '\d+\.\d+'
```

Qt 5.14.2 was used for that guide, in this case minimum supported macOS version is 10.13.

### Building OpenSSL

OpenSSL in Qt' dependency, so build it first. In this guide it is built as shared library.

At the time of writing OpenSSL 1.1.1g was the latest, so source directory is `openssl-1.1.1g`. Go to it

```sh
cd openssl-1.1.1g
```

and issue configuration command:

```sh
./config no-comp no-deprecated no-dynamic-engine no-tests no-zlib --openssldir=/etc/ssl --prefix="$HOME/tmp/qbt/ext" -mmacosx-version-min=10.13
```

Please note `-mmacosx-version-min` option at the end of line, this is NOT some OpenSSL configure script option, this option is passed directly to compiler, it is important to place it last. Its value must be the value that was found in Qt sources.

To start build process just run `make`:

```sh
make -j$(sysctl -n hw.ncpu)
```

Build process takes few minutes (3-5), when it finishes, install everything compiled:

```sh
make install_sw
```

Please note '_sw' suffix, it used just to install a subset of available stuff, this is sufficient for successful build.

### Building Qt

The preferred way to build Qt is 'out of tree build'. So create separate build directory at the same level as source directory and go into it:

```sh
mkdir build-qt && cd build-qt
```

Issue Qt configuration command:

```sh
../qt5/configure -prefix "$HOME/tmp/qbt/ext" -opensource -confirm-license -release -appstore-compliant -c++std c++14 -no-pch -I "$HOME/tmp/qbt/ext/include" -L "$HOME/tmp/qbt/ext/lib" -make libs -no-compile-examples -no-dbus -no-icu -qt-pcre -system-zlib -ssl -openssl-linked -no-cups -qt-libpng -qt-libjpeg -no-feature-testlib -no-feature-sql -no-feature-concurrent
```

This configures Qt as shared library (.framework in case of macOS) with no any debug info included.

Note `-I` and `-L` options in that line, they are required to allow Qt' build system to find OpenSSL (it is an optional dependency, but it is required in case of qBittorrent). If you change paths used in this guide to your own, please make sure that these options have correct values, also please set `-prefix` value to corresponding path too.

The rest of options mostly to minimize the scope of building stuff and decrease build time.

Configuration process takes a few minutes, some required tools are build during it.

When configuration process is finished, build can be started:

```sh
make -j$(sysctl -n hw.ncpu)
```

This step is most time consuming of all that guide, build process takes about 30 minutes.

When it finishes, just install Qt as any other library:

```sh
make install
```

### Building Boost

Actually no Boost binaries are required, libtorrent requires Boost.System and it became header-only since Boost 1.70 (or even 1.69), boost_system library is just a stub left for compatibility, but a lot of tools/scripts rely on it when detecting the presence of Boost. Moreover, there is no option to build header-only version of Boost. Of course, 'stage' version can be used without any building, but it is not suitable for usage with cmake - it doesn't have files allowing cmake to detect it. Such files are generated only during installation process. So build only boost_system library regardless this is just a stub and will not be used:

```sh
cd boost_1_73_0
./bootstrap.sh
./b2 --prefix="$HOME/tmp/qbt/ext" --with-system variant=release link=static cxxflags="-std=c++14 -mmacosx-version-min=10.13" install
```

This produces static version of boost_system library with no debug information. Custom `cxxflags` are left for historical reasons, when Boost.System was not header-only library and apps using it must link boost_system library.

### Building libtorrent

libtorrent provides few build systems to choose from to build it. Unfortunately, convenient and easy to use GNU autotools option is not available on macOS, so let's use cmake.

cmake supports build in separate directory, usually subdirectory is used for it, so create such directory and run cmake in it:

```sh
cd libtorrent
mkdir build && cd build
```

Configure libtorrent as static library with all other options set to default:

```sh
/Applications/CMake.app/Contents/bin/cmake -DCMAKE_PREFIX_PATH="$HOME/tmp/qbt/ext" -DCMAKE_CXX_STANDARD=14 -DCMAKE_CXX_EXTENSIONS=OFF -DCMAKE_OSX_DEPLOYMENT_TARGET=10.13 -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=OFF -DCMAKE_INSTALL_PREFIX="$HOME/tmp/qbt/ext" ..
```

Few important things to note in this line are C++ standard version and minimum supported macOS version (`CMAKE_CXX_STANDARD` and `CMAKE_OSX_DEPLOYMENT_TARGET` options), this is required for successful linkage.

Value of `CMAKE_PREFIX_PATH` is also important, it tells cmake where to find any required dependency. So adjust it in case of using custom paths.

cmake just performs configuration steps and generates platform-specific makefile, so build and install it as usual (remember, you are still in 'build' directory):

```sh
make -j$(sysctl -n hw.ncpu)
make install
```

Build process takes about 5-7 minutes.

### Building qBittorrent

Like libtorrent, qBittorrent also provides few build systems to choose from to build it. GNU autotools is not available on macOS, qmake requires some additional work to setup project, so let's use cmake again.

As for libttorrent, let's create a separate build directory:

```sh
mkdir build && cd build
```

Now everything is ready to issue cmake (from build directory).

qBittorrent 4.3.x series and master branch requires C++17 features available only starting from macOS 10.14. This requirement must be explicitly expressed as cmake arguments.

```sh
/Applications/CMake.app/Contents/bin/cmake -DCMAKE_PREFIX_PATH="$HOME/tmp/qbt/ext" -DCMAKE_CXX_STANDARD=17 -DCMAKE_CXX_EXTENSIONS=OFF -DCMAKE_OSX_DEPLOYMENT_TARGET=10.14 -DCMAKE_BUILD_TYPE=Release ..
```

This configures qBittorrent with all default build options. Only one important option to note - `CMAKE_PREFIX_PATH`, tells cmake where to find dependencies.

Now run build as usual:

```sh
make -j$(sysctl -n hw.ncpu)
```

Build process takes about 10 minutes, and as result you will get `qbittorrent.app` in the build directory (where cmake and make were run). But this app bundle is incomplete - Qt runtime must be integrated into it. This can be achieved with special Qt tool called `macdeployqt`, it even can produce dmg image ready for redistribute.

```sh
$HOME/tmp/qbt/ext/bin/macdeployqt qbittorrent.app -dmg
```

Now `qbittorrent.app` bundle is ready to use. Also `qbittorrent.dmg` is created (with the same app inside). Drop `-dmg` option if don't want it.

That's all! Now you have your own qBittorrent build!

You can use [this script][macos-build-script] to build qBittorrent master branch in "fully automatic mode". Just launch it and wait. This script is a little bit tricky, but it is not too complex to understand. Moreover, it is pretty well documented, sometimes it contains very detailed explanations. Initially, I created it just to fulfil my own needs, but later it was published due to often user requests. It can be a good starting point for creating your own build script serving your own needs/requirements/preferences.

Next sections for advanced users, mostly for anyone who want to develop (or just build) qBittorrent using qmake.

## Building with qmake

First of all, this section for advanced or just very curious users, who want to know how to build qBittorrent using qmake.

As was written before, qBittorrent supports several build systems, one of them is Qt native build system - qmake.

Unfortunately, it is not so convenient as cmake or autotools in case of qBittorrent (due to complex dependencies), but qBittorrent authors envisaged it by providing the way for user-specific configuration (e.g. paths to dependencies) as separate file called `conf.pri`.

Except paths to dependencies, this file also should contain defines describing libtorrent configuration (libtorrent is very sensitive to it, if it itself and app are built with different set of defines it won't link).

If you just followed this guide to build all required dependencies, next `conf.pri` is suitable for building qBittorrent (with no any adjustments). In other cases adjustments are required (at least for paths).

```qmake
INCLUDEPATH += $$PWD/../../ext/include

LIBS += -framework CoreFoundation -framework SystemConfiguration
LIBS += -L$$PWD/../../ext/lib -ltorrent-rasterbar -lssl -lcrypto
LIBS += -liconv -lz

DEFINES += BOOST_ALL_NO_LIB
DEFINES += BOOST_ASIO_ENABLE_CANCELIO
DEFINES += TORRENT_USE_ICONV
DEFINES += TORRENT_USE_LIBCRYPTO
DEFINES += TORRENT_USE_OPENSSL

CONFIG -= silent
CONFIG += strict_c++ c++17
```

Most of its content is libtorrent-related stuff (libtorrent dependencies and mentioned above defines). Line `CONFIG -= silent` is just my preference, it may be useful in development process, it disables "silent build", i.e. all build commands are fully shown with all arguments.

In case if you built libtorrent with your own options (for example with disabled deprecated stuff), you have to adjust defines set. There are some tips on how to find them.

libtorrent defines used for building it can be found in few places: .pc file (used by pkg-config tool on Linux) and cmake config files. The set of defines is the same in these files.

.pc file is regenerated regardless than pkg-config can't used on macOS (at least out of the box), this file can be found at `$HOME/tmp/qbt/ext/lib/pkgconfig/libtorrent-rasterbar.pc` (adjust path to yours if required). You are interested in line staring with "Cflags".

Mentioned cmake config file can be found at `$HOME/tmp/qbt/ext/lib/cmake/LibtorrentRasterbar/LibtorrentRasterbarTargets.cmake` (adjust path to yours if required). You are interested in line starting with "set_target_properties".

I recommend to use cmake file, it looks more reliable, but pkg-config is easy to read.

`conf.pri` must be placed to sources root, then qBittorrent can be built using qmake from command line or by opening project in [QtCreator](#building-with-qtcreator).

To build qBittorrent using qmake from command line, just issue next few commands (assuming you are in `$HOME/tmp/qbt/src`):

```sh
mkdir build-qbt && cd build-qbt
$HOME/tmp/qbt/ext/bin/qmake ../qBittorrent/qbittorrent.pro
make -j$(sysctl -n hw.ncpu)
```

First line just creates out of tree build directory, second runs qmake to generate makefile (release configuration is used by default), and the last just runs make to build everything.

As result, you will get `qbittorrent.app` under `src` subdirectory in your build directory (not like in case of cmake). This bundle is also incomplete, and Qt runtime must be deployed for it:

```sh
cd src          # you are still in build directory
$HOME/tmp/qbt/ext/bin/macdeployqt qbittorrent.app -dmg
```

Now `qbittorrent.app` build is ready to use. Again, if you don't want .dmg image, just drop `-dmg` option, it is used just to show that `macdeployqt` has it.

### Building with QtCreator

To build qBittorrent using QtCreator few additional steps are required.

First of all, custom Qt Kit should be added. Just open QtCreator settings dialog, [add Qt compiled for qBittorrent][adding-qt-version] and then [create new kit][adding-qt-kits] that uses it.

Next important thing that some project files must be edited, otherwise it could be impossible to launch qBittorrent from QtCreator. Just **delete/comment** next line in `macxconf.pri` (in qBittorrent sources root):

```qmake
QMAKE_BUNDLE_DATA += qt_conf
```

Now everything is ready for development or just building.

One more note for new contributors: for a better debugging experience, build at least Qt with debug symbols. Just drop `-release` option from suggested configuration command for it. Please note build time will be approximately twice longer.

[xcode-appstore]: https://apps.apple.com/us/app/xcode/id497799835
[cmake-off-site]: https://cmake.org/download/

[openssl-site]: https://www.openssl.org/source/
[boost-site]: https://www.boost.org/
[libtorrent-repo]: https://github.com/arvidn/libtorrent
[qbittorrent-repo]: https://github.com/qbittorrent/qBittorrent

[macos-build-script]: https://gist.github.com/Kolcha/3ccd533123b773ba110b8fd778b1c2bf

[adding-qt-version]: https://doc.qt.io/qtcreator/creator-project-qmake.html
[adding-qt-kits]: https://doc.qt.io/qtcreator/creator-targets.html
