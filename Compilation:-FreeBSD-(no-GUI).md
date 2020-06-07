## qBittorrent-nox compilation on FreeBSD (with dependencies as system packages)

1. Install required libraries from repository  
> - pkgconf 
> - qt5-core
> - qt5-network
> - qt5-svg
> - qt5-linguisttools
> - qt5-buildtools
> - qt5-qmake
> - boost-all
> - libtorrent-rasterbar
>
> **Copy-paste command**
> ```sh
> pkg install pkgconf qt5-core qt5-network qt5-svg qt5-linguisttools qt5-buildtools qt5-qmake boost-all libtorrent-rasterbar
> ```
2. Execute configure script in extracted qBittorrent source or cloned git repository
> ```sh
> ./configure --disable-gui --prefix=/usr
> ```
3. Build qBittorrent (use parameter `-j <x>` for multi-threaded compilation)
> ```sh
> make
> ```
4. (Optionally) you can install qBittorrent in the system PATH (root privileges required)  
**_You can also use `qbittorrent-nox` binary directly without installation (`./src/qbittorrent-nox`)_**
> ```sh
> make install
> ```