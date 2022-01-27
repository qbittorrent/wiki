# Introduction

qBittorrent is available for Linux, Windows and macOS.
It can be compiled from source for any of those platforms, but binary packages are provided for convenience.

## Linux

qBittorrent binary packages should be available in your distro's repositories. Both the `qbittorrent` (qBittorrent with GUI) and `qbittorrent-nox` ("no X", for headless server environments) packages should be available.

It is recommended to always use the latest version, and new issue reports should be filed against the latest stable version (or a more recent `master` commit) only. However, some distributions do not always provide the latest packages. In this case, you should either:
* Compile from source (https://github.com/qbittorrent/qBittorrent/wiki#compilation)
* Use an official alternative binary repository (such as the official PPAs for Ubuntu)
* Use an alternative unofficial binary repository (at your own risk)

Currently, two official PPAs are maintained for Ubuntu:
* The latest stable version: https://launchpad.net/~qbittorrent-team/+archive/ubuntu/qbittorrent-stable
* The latest commits to `master`: https://launchpad.net/~qbittorrent-team/+archive/ubuntu/qbittorrent-unstable

qBittorrent is also available semi-officially on FlatHub:
* FlatHub home: https://flathub.org/apps/details/org.qbittorrent.qBittorrent
* Source repository: https://github.com/flathub/org.qbittorrent.qBittorrent

For CentOS, Fedora and RHEL, enable the Extra Packages for Enterprise Linux (EPEL) repository (epel-release) to install qbittorrent or qbittorrent-nox.

Unofficial nightly builds for Debian (from [Kolcha](https://github.com/Kolcha)):
* Repository: https://software.opensuse.org//download.html?project=home%3Anikoneko%3Atest&package=qbittorrent
* Discussion thread: https://github.com/qbittorrent/qBittorrent/issues/12616

## Windows and macOS

The **only** official source for Windows and macOS release builds is this page (and the links within it): https://www.qbittorrent.org/download.php

Experimental development builds are available here: https://github.com/qbittorrent/qBittorrent/actions
When reporting bugs against these versions, be sure to mention which specific one you used and its commit ID (which is visible in the top left corner after clicking in a specific run).

Use any other builds at your own risk.
Issue reports from using other builds such as PortableApps and the like are not supported.

You can also compile from source if you wish, by following the relevant guides listed under https://github.com/qbittorrent/qBittorrent/wiki#compilation
