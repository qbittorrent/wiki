# Building system
A full installation of Xcode.app is required to compile this software.<br/>
Installing just the Command Line Tools is not sufficient.<br/>
Xcode can be installed from the [App Store](https://www.apple.com/appstore).<br/>

After installing Xcode you need to do bellow. See [this discussion](http://stackoverflow.com/questions/33728905/qt-creator-project-error-xcode-not-set-up-properly-you-may-need-to-confirm-t).

`sudo xcode-select -switch /Applications/Xcode.app/Contents/Developer`

`sudo ln -s /Applications/Xcode.app/Contents/Developer/usr/bin/xcodebuild /Applications/Xcode.app/Contents/Developer/usr/bin/xcrun`

# Install Homebrew
See [Install Homebrew](http://brew.sh/).

# Install tools and dependencies
`brew install pkg-config autoconf automake libtool openssl boost`

# Install ibtorrent-rasterbar from source
`git clone https://github.com/arvidn/libtorrent.git`

`cd libtorrent`

`git checkout RC_1_0`

Edit the src/Makefile.am file. Find the `libtorrent_rasterbar_la_LIBADD =` line and add `@OPENSSL_LDFLAGS@` before `@OPENSSL_LIBS@`.<br/>
`sed -i “” -e "s/^\(libtorrent_rasterbar_la_LIBADD\)\(.*\)\(@OPENSSL_LIBS@\)/\1\2@OPENSSL_LDFLAGS@  \3/" src/Makefile.am`

`./autotool.sh`

`CXXFLAGS=-std=c++11 ./configure --disable-dependency-tracking --disable-silent-rules --enable-encryption --prefix=/usr/local --with-boost=/usr/local/opt/boost --with-openssl=/usr/local/opt/openssl`<br/>

`make && make install`

# Install qt5 from source
`curl -L -O https://download.qt.io/official_releases/qt/5.7/5.7.0/single/qt-everywhere-opensource-src-5.7.0.tar.xz`

`tar xvf qt-everywhere-opensource-src-5.7.0.tar.xz`

`cd qt-everywhere-opensource-src-5.7.0`

Apply [this patch](https://github.com/Homebrew/homebrew-core/issues/3219#issuecomment-235820697).<br/>
`curl https://gist.githubusercontent.com/okeatime/dc2f7dabd9321e8b57cdb27a096e4058/raw/72dc3618423b1e9876d3d4b94412f977b9a2f33e/macdeployqt.patch | patch -p1`

`OPENSSL_LIBS='-L/usr/local/opt/openssl/lib -lssl -lcrypto' ./configure -prefix /usr/local/qt5.7.0 -I/usr/local/opt/openssl/include -no-rpath -opensource -confirm-license -release -openssl-linked -no-securetransport -make libs -make tools -nomake examples -nomake tests -skip qt3d -skip qtactiveqt -skip qtandroidextras -skip qtcanvas3d -skip qtdeclarative -skip qtdoc -skip qtgraphicaleffects -skip qtimageformats -skip qtlocation -skip qtmacextras -skip qtmultimedia -skip qtquickcontrols -skip qtscript -skip qtsensors -skip qtserialport -skip qtsvg -skip qtwayland -skip qtwebchannel -skip qtwebsockets -skip qtwinextras -skip qtx11extras -skip qtxmlpatterns -skip qtwebview -skip qtwebengine -skip qtconnectivity -v`

`make && make install`

# Download qBittorrent source
 
 1. Download main source from [qBittorrent's](http://www.qbittorrent.org/download.php) download page and extract the tarball or grab the HEAD from GitHub (`git clone https://github.com/qbittorrent/qBittorrent.git`).
 2. Optional: Download the [geoip dat](http://geolite.maxmind.com/download/geoip/database/GeoLiteCountry/GeoIP.dat.gz) file and extract it to qbittorrent's src/gui/geoip folder using gzip or a similar tool.

# Compilation

 1. Set environment variable: `export QT_QMAKE=/usr/local/qt5.7.0/bin`
 2. Configuration: `./configure --disable-qt-dbus`
 3. Compilation: `make -j4` where 4 is your number of cores
 4. Packaging: `$QT_QMAKE/macdeployqt src/qbittorrent.app` (may require sudo)
 5. Or packaging and create DMG: `$QT_QMAKE/macdeployqt src/qbittorrent.app -dmg` (may require sudo)

# Optionally install python for the search function
You can choose python2 or python3.

`brew install python`<br/>
or<br/>
`brew install python3`
