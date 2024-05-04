# Introduction

**NOTE:** for qBittorrent revisions older than `63ff5e3` (2020-09-19), use the [legacy guide](https://github.com/qbittorrent/qBittorrent/wiki/Compilation-Debian-and-Ubuntu) instead.

This how-to will guide you through compiling qBittorrent from source on Debian, Ubuntu, and other derivative distros.

Only follow this guide if you know what you are doing and this is what you really want.
If you just want the latest version of qBittorrent, use our [stable](https://launchpad.net/~qbittorrent-team/+archive/ubuntu/qbittorrent-stable) or [unstable](https://launchpad.net/~qbittorrent-team/+archive/ubuntu/qbittorrent-unstable) PPAs.

# Install the dependencies

## Boost and other miscellaneous build dependencies

You need to install these packages in order to be able to compile qBittorrent from source:

```bash
sudo apt install build-essential cmake git ninja-build pkg-config libboost-dev libssl-dev zlib1g-dev libgl1-mesa-dev
```

## Qt libraries

qBittorrent uses the Qt framework as the basis for its GUI.

- qBittorrent 4.4.x requires at least Qt 6.5.0.
- At the time of writing, the current `master` branch requires at least Qt 6.5.0.

Many distributions, in particular Debian, Ubuntu (especially LTS releases), and their derivatives don't provide up-to-date Qt packages in their repositories or are very slow in updating them.
In such cases, you must get them from somewhere else, such as the official installer from the [Qt website](https://www.qt.io/download-qt-installer) (unfortunately, this method requires the creation of an account, but you can just use a throwaway email), or a PPA you trust in the case of Ubuntu and other distributions that support that installation method.

For Debian and Ubuntu versions that include sufficiently up-to-date Qt packages, you can just install the following packages from the official repositories:

```bash
sudo apt install --no-install-recommends qt6-base-dev qt6-tools-dev qt6-svg-dev
```

## libtorrent

qBittorrent uses the [libtorrent](https://libtorrent.org/) library by Arvid Norberg (aka `libtorrent-rasterbar`) as the backend.

It is possible to install the `libtorrent` development package directly from the distro's repositories:

```bash
sudo apt install libtorrent-rasterbar-dev
```

but the version may not be the most recent or exactly the one you want.

Alternatively, you can compile `libtorrent` yourself. qBittorrent 4.2.x and above requires the  1.2.x series(*).
qBittorrent 4.4.x requires minimum libtorrent 1.2.14 or 2.0.4.

```bash
git clone --recurse-submodules https://github.com/arvidn/libtorrent.git
cd libtorrent
git checkout RC_2_0 # or a 2.0.x tag
cmake -B build -DCMAKE_BUILD_TYPE=RelWithDebInfo -DCMAKE_INSTALL_PREFIX=/usr/local
cmake --build build
sudo cmake --install build
```

The install step will install libtorrent to the chosen prefix (`/usr/local`, in this case), and generate an `install_manifest.txt` file in the build folder that can later be used to uninstall all installed files with `sudo xargs rm < install_manifest.txt`.

For more information on building and installing libtorrent (such as the available build configuration options), read [the documentation](https://www.libtorrent.org/building.html).

(*)Technically, 4.2.x releases up to, and including, 4.2.5, actually support the 1.1.x `libtorrent` series as well, but it is just life support and not properly tested/developed.

## Runtime-only dependencies

qBittorrent has a runtime-only dependency on Python **3** for its search engine functionality:

```bash
sudo apt install python3
```

# Build qBittorrent

Download and extract a `.tar` archive from [the GitHub releases page](https://github.com/qbittorrent/qBittorrent/releases) or clone the repository and checkout the branch/tag of your choice.

Then, configure and build with CMake:

```bash
cmake -G "Ninja" -B build -DCMAKE_BUILD_TYPE=RelWithDebInfo -DCMAKE_INSTALL_PREFIX=/usr/local
cmake --build build
```

Check out the [common information](https://github.com/qbittorrent/qBittorrent/wiki/Compilation-with-CMake-common-information) page to learn more about the available build configuration options (for compiling without the GUI, for instance) and CMake itself, if you're new to it.

Once qBittorrent is built, you can run it straight from the build directory. Documentation about running qBittorrent without GUI is available [here](https://github.com/qbittorrent/qBittorrent/wiki/Running-qBittorrent-without-X-server-(WebUI-only)).

## Installing (optional)

To install qBittorrent to the prefix chosen at configure-time:

```
sudo cmake --install build
```

This will generate an `install_manifest.txt` file in the build folder that can later be used to uninstall all the installed files with `sudo xargs rm < install_manifest.txt`. To override the installation prefix at install-time, pass `--prefix <install_prefix>`.

Patches are welcome to implement Debian package generation with CPack.
