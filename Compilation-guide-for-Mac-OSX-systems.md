# Building system
Make sure your system is running the latest xcode build system with the commandline toolset.

# Dependencies

## Custom
See the [INSTALL](https://github.com/qbittorrent/qBittorrent/blob/master/INSTALL) file for a list of dependencies.


## Using Homebrew
[Install Homebrew](http://brew.sh/)

    brew install qt4 libtorrent-rasterbar

# Download source
 
 1. Download main source from [qBittorrent's](http://www.qbittorrent.org/download.php) download page and extract the tarball or grab the HEAD from GitHub (`git clone https://github.com/qbittorrent/qBittorrent.git`).
 2. Optional: Download the [geoip dat](http://geolite.maxmind.com/download/geoip/database/GeoLiteCountry/GeoIP.dat.gz) file and extract it to qbittorrent's src/gui/geoip folder using gzip or a similar tool.

# Compilation

 1. Configuration `./configure --disable-qt-dbus`
 2. Compilation: `make -j4` where 4 is your number of cores
 3. Packaging: `macdeployqt src/qbittorrent.app` (may require sudo)
 4. Create DMG: `macdeployqt src/qbittorrent.app -dmg` (may require sudo)

