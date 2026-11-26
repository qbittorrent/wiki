# Introduction

This is a short guide for building qBittorrent 5.0.0+ with Qt6 on Debian.

# Install the dependencies

*NOTE:* `Qt6 libboost libtorrent-rasterbar` on current Debian stable(bookworm) does not meet minimal requirements.
You can switch to Debian testing(trixie) or build dependencies from source.

## Boost and other miscellaneous build dependencies

You need to install these packages in order to be able to compile qBittorrent from source:

```bash
sudo apt install build-essential cmake git ninja-build pkg-config libboost-dev libssl-dev zlib1g-dev libgl1-mesa-dev
```

## Qt libraries

qBittorrent uses the Qt framework as the basis for its GUI. qBittorrent 5.0.0+ requires Qt6.

Install the following packages from the official repositories:

```bash
sudo apt install qt6-base-dev qt6-base-private-dev qt6-svg-dev qt6-tools-dev qt6-tools-private-dev
```

## libtorrent

qBittorrent uses the [libtorrent](https://libtorrent.org/) library by Arvid Norberg (aka `libtorrent-rasterbar`) as the backend.

It is possible to install the `libtorrent` development package directly from the distro's repositories:

```bash
sudo apt install libtorrent-rasterbar-dev
```

## Runtime-only dependencies

qBittorrent has a runtime-only dependency on Python **3** for its search engine functionality:

```bash
sudo apt install python3
```

# Build qBittorrent

Configure and build with CMake:

```bash
cmake -G "Ninja" -B build -DCMAKE_BUILD_TYPE=RelWithDebInfo -DCMAKE_INSTALL_PREFIX=/usr/local
cmake --build build
```
