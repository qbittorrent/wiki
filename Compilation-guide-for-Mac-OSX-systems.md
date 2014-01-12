# Building system
Make sure your system is running the latest xcode build system with the commandline toolset.

# Dependencies

## Custom
See the [INSTALL](https://github.com/qbittorrent/qBittorrent/blob/master/INSTALL) file for a list of dependencies.

Adjust paths in `macxconf.pri` accordingly. See example using Homebrew

## Using MacPorts
[Install MacPorts](http://www.macports.org/install.php)

    sudo port install qt4-mac libtorrent-rasterbar

## Using Homebrew
[Install Homebrew](http://brew.sh/)

    brew install qt4 libtorrent-rasterbar

Adjust paths in `macxconf.pri` from

    INCLUDEPATH += /usr/include/openssl /usr/include /opt/local/include/boost /opt/local/include
    LIBS += -L/opt/local/lib

to

    INCLUDEPATH += /usr/include/openssl /usr/include /usr/local/include/boost /usr/local/include
    LIBS += -L/usr/local/lib

# Download source
 
 1. Download main source from [qBittorrent's](http://www.qbittorrent.org/download.php) download page, extract the tarball.
 2. Optional: Download the [geoip dat](http://geolite.maxmind.com/download/geoip/database/GeoLiteCountry/GeoIP.dat.gz) file.and extract it to qbittorrents src/geoip folder.

# Compilation

 1. Configuration: `qmake qbittorrent.pro`  If this fails, try using `./configure`
 2. Compilation: `make -j4` where 4 is your number of cores
 3. Packaging: `macdeployqt src/qbittorrent.app` (may require sudo)
 4. Create DMG: `macdeployqt src/qbittorrent.app -dmg` (may require sudo)
