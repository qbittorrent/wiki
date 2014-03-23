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

# Compilation with MacPorts

-- No Xcode installation is required (ignore it if you have already installed).  
Download and install Command Line Tools kit available at: https://developer.apple.com/downloads/index.action  
Free registration of developer account is required for this off-line method.  

Note Mac OS 10.9.x is bundled with xcode-select command and any invocation of developer's commands in Terminal application such as "xcode-select --install" or "gcc --version" will cause a popup on display to request for online installation of the tool kit from Apple update servers, just click on "Install" button to proceed with this alternative on-line method.  

-- Use xcode-select command to determine active developer directory: xcode-select --print-path  
Either "/Applications/XCode.app/Contents/Developer" for Xcode installation or "/Library/Developer/CommandLineTools" for Command Line Tools kit installation will be displayed.  
Set it to newly installed directory /Library/Developer/CommandLineTools with following command:  
xcode-select --switch /Library/Developer/CommandLineTools  
To remove Xcode installation to recover huge disk space, simply drag the Xcode application to trash and empty trash as Apple has packaged everything within this self-contained application since release of Xcode 4.3.

-- Install MacPorts according to instructions given in: https://guide.macports.org/index.html  
It is not necessary to install X11 package (X Windows applications) for qbittorrent application. The easier way is to download installer package MacPorts-2.2.1-10.9-Mavericks.pkg and install it like any Mac OS X applications. Use the default /opt/local root directory for installations of all Unix packages through MacPorts. Take note of proper settings of environment variables PATH (for executable binary files) and MANPATH (for manual page documentation) in your shell after installation of MacPorts.

-- Install all Unix packages required by qbittorrent application:  
sudo port install qt4-mac libtorrent-rasterbar pkgconfig  
Administrator password required by the command prefix sudo. All together there will be 33 Unix packages being installed, so be patient as fetching, building and installation of these packages take time particularly if your processor or Internet connection is slow. To confirm these packages have been installed with port command:  
  
% port installed  
The following ports are currently installed:  
  autoconf @2.69_2 (active)  
  automake @1.14.1_1 (active)  
  boost @1.55.0_2+no_single+no_static+python27 (active)  
  bzip2 @1.0.6_0 (active)  
  db46 @4.6.21_9 (active)  
  db_select @0.1_2 (active)  
  dbus @1.6.12_0+startupitem (active)  
  expat @2.1.0_0 (active)  
  gdbm @1.11_0 (active)  
  gettext @0.18.3.2_0 (active)  
  icu @51.2_1 (active)  
  jpeg @9a_1 (active)  
  lcms @1.19_5 (active)  
  libedit @20121213-3.0_0 (active)  
  libgeoip @1.5.1_0 (active)  
  libiconv @1.14_0 (active)  
  libmng @1.0.10_3 (active)  
  libpng @1.6.9_1 (active)  
  libtool @2.4.2_3 (active)  
  libtorrent-rasterbar @0.16.13_0+python27 (active)  
  m4 @1.4.16_0 (active)  
  ncurses @5.9_2 (active)  
  openssl @1.0.1f_0 (active)  
  perl5 @5.12.4_0+perl5_16 (active)  
  perl5.16 @5.16.1_3 (active)  
  pkgconfig @0.28_0 (active)  
  python27 @2.7.6_0 (active)  
  python_select @0.3_3 (active)  
  qt4-mac @4.8.5_1 (active)  
  sqlite3 @3.8.3.1_0 (active)  
  tiff @4.0.3_2 (active)  
  xz @5.0.5_0 (active)  
  zlib @1.2.8_0 (active)  

-- Download qbittorrent 3.1.9 source compressed archive and GeoIP.dat.gz compressed data file and place them in a folder under your home directory:  
% ls ~/project/qbittorrent  
qbittorrent-3.1.9.tar.gz   GeoP.dat.gz  
% cd ~/project/qbittorrent  

-- Unpack qbittorrent's source code and GeoIP.dat data file into proper locations as follows:  
% gzip -cd qbittorrent-3.1.9.tar.gz | tar -xvf -  
% gzip -cd GeoIP.dat.gz > qbittorrent-3.1.9/src/geoip/GeoIP.dat  

-- Modify path settings in configuration file macxconf.pri (backup file: macxconf.pri.backup)  
% sed -i '.backup' '/^INCLUDEPATH/,/^LIBS/ s:/opt/local:/usr/local:g' qbittorrent-3.1.9/macxconf.pri  

-- Change directory to create Make file and initiate compilation, the resultant qbittorrent application file will be located in src directory.  
% cd qbittorrent-3.1.9  
% qmake abittorrent.pro  
% make  

-- Optional steps to package and create disk image file for future distribution:  
% sudo macdeployqt src/qbittorrent.app  
% sudo macdeployqt src/qbittorrent.app -dmg  

-- Clean up directory to conserve disk space as last step:  
% make clean  
