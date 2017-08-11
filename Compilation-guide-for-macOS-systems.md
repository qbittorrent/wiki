# Install Xcode
A full installation of Xcode.app is required to compile qt5.<br/>
Xcode can be installed from the [App Store](http://www.apple.com/osx/apps/app-store/).<br/>

After installing Xcode you need to do below. See [this discussion](http://stackoverflow.com/questions/33728905/qt-creator-project-error-xcode-not-set-up-properly-you-may-need-to-confirm-t).

`sudo xcode-select -switch /Applications/Xcode.app/Contents/Developer`

`(cd /Applications/Xcode.app/Contents/Developer/usr/bin; sudo ln -s xcodebuild xcrun)`

# Install Homebrew
See [Install Homebrew](http://brew.sh/).

# Install tools and dependencies
`brew install pkg-config autoconf automake libtool openssl boost`

If you want to use libtorrent 1.1.*, you need to do<br/>
`brew link libtool`

# Install [libtorrent-rasterbar](https://github.com/arvidn/libtorrent) from source
`git clone https://github.com/arvidn/libtorrent.git`

`cd libtorrent`

libtorrent 1.0.* <br/>
`git checkout RC_1_0`<br/>
Or libtorrent 1.1.* <br/>
`git checkout RC_1_1`

`./autotool.sh`

Edit the src/Makefile.in file. Find the libtorrent_rasterbar_la_LIBADD = line and add `@OPENSSL_LDFLAGS@` before `@OPENSSL_LIBS@`.<br/>
`sed -i “” -e "s/^\(libtorrent_rasterbar_la_LIBADD\)\(.*\)\(@OPENSSL_LIBS@\)/\1\2@OPENSSL_LDFLAGS@ \3/" src/Makefile.in`

`./configure --disable-debug --disable-dependency-tracking --disable-silent-rules --enable-encryption --prefix=/usr/local --with-boost=/usr/local/opt/boost --with-openssl=/usr/local/opt/openssl CXXFLAGS=-std=c++11`<br/>

`make -j4` where 4 is your number of cores

`make install`

# Install qt5 from source
`curl -L -O https://download.qt.io/official_releases/qt/5.7/5.7.1/single/qt-everywhere-opensource-src-5.7.1.tar.xz`

`tar xvf qt-everywhere-opensource-src-5.7.1.tar.xz`

`cd qt-everywhere-opensource-src-5.7.1`

Apply [this patch](https://github.com/Homebrew/homebrew-core/issues/3219#issuecomment-235820697).<br/>
`curl https://github.com/okeatime/qBittorrent/releases/download/depend.tar.ball/macdeployqt.patch | patch -p1`

`OPENSSL_LIBS='-L/usr/local/opt/openssl/lib -lssl -lcrypto' ./configure -prefix /usr/local/qt5.7.1 -I/usr/local/opt/openssl/include -no-rpath -opensource -confirm-license -release -openssl-linked -no-securetransport -make libs -make tools -nomake examples -nomake tests -skip qt3d -skip qtactiveqt -skip qtandroidextras -skip qtcanvas3d -skip qtdeclarative -skip qtdoc -skip qtgraphicaleffects -skip qtimageformats -skip qtlocation -skip qtmacextras -skip qtmultimedia -skip qtquickcontrols -skip qtscript -skip qtsensors -skip qtserialport -skip qtsvg -skip qtwayland -skip qtwebchannel -skip qtwebsockets -skip qtwinextras -skip qtx11extras -skip qtxmlpatterns -skip qtwebview -skip qtwebengine -skip qtconnectivity -v`

`make -j4 && make install`

# Download qBittorrent source
 
 1. Download main source from [qBittorrent's](http://www.qbittorrent.org/download.php) download page and extract the tarball or grab the HEAD from GitHub (`git clone https://github.com/qbittorrent/qBittorrent.git`).
 2. Optional: Download the [geoip dat](http://geolite.maxmind.com/download/geoip/database/GeoLiteCountry/GeoIP.dat.gz) file and extract it to qbittorrent's src/gui/geoip folder using gzip or a similar tool.

# Compilation
## With Autotools
* Set environment variable: `export QT_QMAKE=/usr/local/qt5.7.1/bin`
* Configuration: `./configure`
* Compilation: `make -j4`
* Packaging: `$QT_QMAKE/macdeployqt src/qbittorrent.app` (may require sudo)
* Or packaging and create DMG: `$QT_QMAKE/macdeployqt src/qbittorrent.app -dmg` (may require sudo)

## With CMake
* `mkdir build`
* `cd build`
* `OPENSSL_ROOT_DIR=/usr/local/opt/openssl Qt5_DIR=/usr/local/qt5.7.1/lib/cmake/Qt5 cmake -DDBUS=OFF ..`
* `make -j4`
* `/usr/local/qt5.7.1/bin/macdeployqt src/app/qbittorrent.app`

# Optionally install python for the search function
You can choose python2 or python3.

`brew install python`<br/>
or<br/>
`brew install python3`