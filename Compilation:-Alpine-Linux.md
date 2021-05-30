# Alpine Linux build instructions

This guide will help you install qBittorrent on Alpine Linux using a combination of packages installed using `apk` and dependencies built using `cmake`.

ℹ️ This guide should work in any [Alpine arch](http://dl-cdn.alpinelinux.org/alpine/latest-stable/main) as long as the required [packages dependencies](https://pkgs.alpinelinux.org) are available to install for that arch. This guide was tested on `x86_64`

ℹ️ This guide will install qBittorrent and dependencies using shared libraries. At the point of writing this guide the latest version of Alpine is `3.13`

The majority of these commands are copy and paste but some can be modified. For example, in the libtorrent section there is a choice between libtorrent `v1` or `v2`. Check the notes and commented commands for more information.

Check out the [common information](https://github.com/qbittorrent/qBittorrent/wiki/Compilation-with-CMake:-common-information) page to learn more about the available build configuration options for qBittorrent (for example, to compile qBittorrent without the GUI) and also CMake itself, if you're new to it.

## Build dependencies

These are the build dependencies we need to install using `apk`

```ash
apk add autoconf automake build-base cmake curl git libtool linux-headers perl pkgconf python3 python3-dev re2c tar
```

## Application dependencies

These are application dependencies we need to install using `apk`

ℹ️ The `iconv` libraries needed for `libtorrent` v1.2 are built into `/usr/lib/libc.so` so we do not need to install `iconv` as a dependency.

```ash
apk add icu-dev libexecinfo-dev openssl-dev qt5-qtbase-dev qt5-qttools-dev zlib-dev qt5-qtsvg-dev
```

## Ninja build

⚠️ There is no `apk` version for Alpine `3.12` or greater so we need to build it. It won't take long.

```bash
git clone --shallow-submodules --recurse-submodules https://github.com/ninja-build/ninja.git ~/ninja && cd ~/ninja
git checkout "$(git tag -l --sort=-v:refname "v*" | head -n 1)"
cmake -Wno-dev -B build \
	-D CMAKE_CXX_STANDARD=17 \
	-D CMAKE_INSTALL_PREFIX="/usr/local"
cmake --build build
cmake --install build
```

## Boost build files

ℹ️  All we need to bootstrap boost is to download and extract the files. There is nothing to build at this step.

Download and extract the boost files.

```bash
curl -sNLk https://boostorg.jfrog.io/artifactory/main/release/1.76.0/source/boost_1_76_0.tar.gz -o "$HOME/boost_1_76_0.tar.gz"
tar xf "$HOME/boost_1_76_0.tar.gz" -C "$HOME"
```

## Libtorrent

Download and build libtorrent by checking out the `RC_1_2` branch. You can also change the `git checkout` command filter `"v1*"` to `"v2*"` to use the latest version of a specific tag

ℹ️ Any tag can be used to checkout the version you want - https://github.com/arvidn/libtorrent/tags

```bash
git clone --shallow-submodules --recurse-submodules https://github.com/arvidn/libtorrent.git ~/libtorrent && cd ~/libtorrent
# git checkout "$(git tag -l --sort=-v:refname "v2*" | head -n 1)" # always checkout the latest release of libtorrent v2
git checkout "$(git tag -l --sort=-v:refname "v1*" | head -n 1)" # always checkout the latest release of libtorrent v1
cmake -Wno-dev -G Ninja -B build \
    -D CMAKE_BUILD_TYPE="Release" \
    -D CMAKE_CXX_STANDARD=17 \
    -D BOOST_INCLUDEDIR="$HOME/boost_1_76_0/" \
    -D CMAKE_INSTALL_LIBDIR="lib" \
    -D CMAKE_INSTALL_PREFIX="/usr/local"
cmake --build build
cmake --install build
```

## qBittorrent

Build and install qBittorrent

ℹ️ Any tag can be used to checkout the version you want - https://github.com/qbittorrent/qBittorrent/tags

⚠️ You are most likely not using a GUI (desktop) with Alpine, so remember to pass `-D GUI=OFF`.

```bash
git clone --shallow-submodules --recurse-submodules https://github.com/qbittorrent/qBittorrent.git ~/qbittorrent && cd ~/qbittorrent
git checkout "$(git tag -l --sort=-v:refname | head -n 1)" # always checkout the latest release of qbittorrent
cmake -Wno-dev -G Ninja -B build \
    -D CMAKE_BUILD_TYPE="release" \
    -D CMAKE_CXX_STANDARD=17 \
    -D BOOST_INCLUDEDIR="$HOME/boost_1_76_0/" \
    -D CMAKE_CXX_STANDARD_LIBRARIES="/usr/lib/libexecinfo.so" \
    -D CMAKE_INSTALL_PREFIX="/usr/local"
cmake --build build
cmake --install build
```

Tidy up the downloaded build files

```bash
cd && rm -rf qbittorrent libtorrent ninja boost_1_76_0 boost_1_76_0.tar.gz
```

## Post build

You can now run `qbittorrent-no` as it will be in the path (or `qbittorrent`, if you build with the GUI).

```bash
qbittorrent-nox
```