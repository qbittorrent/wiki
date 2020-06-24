# Introduction

qBittorrent is available for Linux, Windows and macOS.
It can be compiled from source for any of those platforms, but binary packages are provided for convenience.

# Linux

qBittorrent binary packages should be available in your distro's repositories. Both the `qbittorrent` (qBittorrent with GUI) and `qbittorrent-nox` ("no X", for headless server environments) packages should be available.

It is recommended to use always use the latest version, and new issue reports should be filed against the latest stable version (or a more recent `master` commit) only. However, some distributions do not always provide the latest packages. In this case, you should either: use an official alternative binary repository (such as the official PPAs for Ubuntu), use an alternative unofficial binary repository (at your own risk) or compile from source.

Currently, two official PPAs are maintained for Ubuntu:

- the latest stable version: https://launchpad.net/~qbittorrent-team/+archive/ubuntu/qbittorrent-stable
- the latest commits to `master`: https://launchpad.net/~qbittorrent-team/+archive/ubuntu/qbittorrent-unstable
 
qBittorrent is also available semi-officially on FlatHub: https://flathub.org/apps/details/org.qbittorrent.qBittorrent

For CentOS, Fedora and RHEL, enable the Extra Packages for Enterprise Linux (EPEL) repository (epel-release) to install qbittorrent or qbittorrent-nox. 

# Windows and macOS

The **only** official source for Windows and macOS builds is this page (and the links within it): https://www.qbittorrent.org/download.php.

Use any other builds at your own risk.
Issue reports from using other builds such as PortableApps and the like are not accepted.

You can also compile from source if you wish, by following the relevant guides:

- Windows: https://github.com/qbittorrent/qBittorrent/wiki/Windows-compilation
- macOS: https://github.com/qbittorrent/qBittorrent/wiki/Compilation-guide-for-macOS-systems
