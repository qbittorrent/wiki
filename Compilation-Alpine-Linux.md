# Alpine Linux build instructions

This guide will help you install qBittorrent on Alpine Linux

> [!NOTE]
>
> - This guide should work in any [Alpine arch](http://dl-cdn.alpinelinux.org/alpine/latest-stable/main) as long as the required [packages dependencies](https://pkgs.alpinelinux.org) are available to install for that arch. This guide was tested on `x86_64`
>
> - This guide will install qBittorrent and dependencies using shared libraries. At the point of writing this guide the latest version of Alpine is `3.20`
>
> - Check out the [common information](https://github.com/qbittorrent/qBittorrent/wiki/Compilation-with-CMake-common-information) page to learn more about the available build configuration options for qBittorrent (for example, to compile qBittorrent without the GUI) and also CMake itself, if you're new to it.

> [!IMPORTANT]
> The majority of these commands are copy and paste but some can be modified. For example, in the libtorrent section there is a choice between libtorrent `v1` or `v2`. Check the notes and commented commands for more information.

## Build dependencies

These are the build dependencies we need to install using `apk`

```shell
apk add build-base cmake curl git linux-headers ninja-build ninja-is-really-ninja python3 re2c tar xz
```

## Application dependencies

These are application dependencies we need to install using `apk`

> [!WARNING]
> If you are using a desktop and want the GUI for qBittorrent, you will need to append the dependency `qt6-qtsvg-dev` to the command below

```shell
apk add icu-dev openssl-dev qt6-qtbase-dev qt6-qttools-dev zlib-dev
```

## Boost build files

> [!TIP]
> This command should provide the latest non beta release info from Github
>
> - `curl -sL https://api.github.com/repos/boostorg/boost/releases | jq -r '.[0].name | select(contains("beta") | not)'`
>
> You can view all the tags for the boost Github repository here:
>
> - https://github.com/boostorg/boost/tags

Download and extract the boost files.

> [!NOTE]
> All we need to bootstrap boost is to download and extract the files. There is nothing to build at this step.

Bootstrap boost dev files

```shell
mkdir -p ~/boost-dev
curl -L https://github.com/boostorg/boost/releases/latest/download/boost-1.86.0-b2-nodocs.tar.xz -o ~/boost.tar.xz
tar xf ~/boost.tar.xz --strip-components=1 -C ~/boost-dev
```

## Libtorrent

Download and build libtorrent by checking out the `RC_1_2` branch. You can also change the `git checkout` command filter `"v1*"` to `"v2*"` to use the latest version of a specific tag

> [!TIP]
> Any tag can be used to checkout the version you want - https://github.com/arvidn/libtorrent/tags

```shell
git clone --shallow-submodules --recurse-submodules https://github.com/arvidn/libtorrent.git ~/libtorrent && cd ~/libtorrent
# git checkout "$(git tag -l --sort=-v:refname | awk '/v2/' | head -1)" # always checkout the latest release of libtorrent v2
git checkout "$(git tag -l --sort=-v:refname | awk '/v1/' | head -1)" # always checkout the latest release of libtorrent v1
cmake -Wno-dev -G Ninja -B build \
    -D CMAKE_BUILD_TYPE="Release" \
    -D CMAKE_CXX_STANDARD=20 \
    -D BOOST_INCLUDEDIR="$HOME/boost-dev/" \
    -D CMAKE_INSTALL_LIBDIR="lib" \
    -D CMAKE_INSTALL_PREFIX="/usr/local"
cmake --build build
cmake --install build
```

## qBittorrent

Build and install qBittorrent

> [!TIP]
> Any tag can be used to checkout the version you want - https://github.com/qbittorrent/qBittorrent/tags

> [!WARNING]
> You are most likely not using a GUI (desktop) with Alpine, so remember we pass `-D GUI=OFF`.
> Removing `-D GUI=OFF` will build the desktop version `qbittorrent` instead of the cli `qbittorrent-nox`
>
> qBittorrent v5 made Qt6 the default and removed support for Qt5 builds.
> To manually set the Qt used for v4 or earlier you can used this `cmake` option `-D QT6=ON` or `-D QT6=OFF`

This command will build qBittorrent `v5` with Libtorrent `v1.2` using `Qt6`, the defaults at the time of this guide.

```shell
git clone --shallow-submodules --recurse-submodules https://github.com/qbittorrent/qBittorrent.git ~/qbittorrent && cd ~/qbittorrent
git checkout "$(git tag -l --sort=-v:refname | awk '!/[0-9][a-zA-Z]/' | head -1)" # always checkout the latest release of qbittorrent
cmake -Wno-dev -G Ninja -B build \
    -D CMAKE_BUILD_TYPE="release" \
    -D CMAKE_CXX_STANDARD=20 \
    -D BOOST_INCLUDEDIR="$HOME/boost-dev/" \
    -D CMAKE_INSTALL_PREFIX="/usr/local" \
    -D GUI=OFF
cmake --build build
cmake --install build
```

## Run the binary

You can now run `qbittorrent` or `qbittorrent-nox` as it will be in the path.

Desktop version: `-D GUI=ON`

```shell
qbittorrent
```

cli version: `-D GUI=OFF`

```shell
qbittorrent-nox
```

## Post installation

Tidy up: Delete the downloaded build files and folders

```shell
cd && rm -rf qbittorrent libtorrent boost-dev ~/boost.tar.xz
```
