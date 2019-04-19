# Install Xcode
A full installation of Xcode.app is required to compile Qt.<br/>
Xcode can be installed from the [App Store](https://www.apple.com/osx/apps/app-store/).<br/>

After installing Xcode you need to do below. See [this discussion](https://stackoverflow.com/questions/33728905/qt-creator-project-error-xcode-not-set-up-properly-you-may-need-to-confirm-t).

```shell
sudo xcode-select -switch /Applications/Xcode.app/Contents/Developer
(cd /Applications/Xcode.app/Contents/Developer/usr/bin; sudo ln -s xcodebuild xcrun)
```

# Install Homebrew
https://brew.sh/

# Install tools and dependencies
```shell
brew install pkg-config autoconf automake libtool openssl boost
```

If you want to use libtorrent 1.1.*, you need to do:
```shell
brew link libtool
```

# Install [libtorrent-rasterbar](https://github.com/arvidn/libtorrent)
## From brew
```shell
brew install libtorrent-rasterbar
```

## From source
```shell
git clone https://github.com/arvidn/libtorrent.git
cd libtorrent

# use libtorrent 1.0.*
git checkout RC_1_0
# or libtorrent 1.1.*
git checkout RC_1_1

./autotool.sh
```

Edit the src/Makefile.in file. Find the libtorrent_rasterbar_la_LIBADD = line and add `@OPENSSL_LDFLAGS@` before `@OPENSSL_LIBS@`:
```shell
sed -i “” -e "s/^\(libtorrent_rasterbar_la_LIBADD\)\(.*\)\(@OPENSSL_LIBS@\)/\1\2@OPENSSL_LDFLAGS@ \3/" src/Makefile.in
```

Then
```shell
./configure --disable-debug --disable-dependency-tracking --disable-silent-rules --enable-encryption --prefix=/usr/local --with-boost=/usr/local/opt/boost --with-openssl=/usr/local/opt/openssl CXXFLAGS=-std=c++11
make -j2  # where 2 is your number of cores
make install
```

# Install Qt
## From brew
```shell
brew install qt
brew link --force qt
```

## Optionally
Instead of force linking qt:
```shell
# use ~/.bashrc if you are using bash
echo export Qt5_DIR=\"/usr/local/opt/qt\" >> ~/.zshrc
source ~/.zshrc
```

## From source
```shell
curl -L -O https://download.qt.io/official_releases/qt/5.7/5.7.1/single/qt-everywhere-opensource-src-5.7.1.tar.xz
tar -xvf qt-everywhere-opensource-src-5.7.1.tar.xz
cd qt-everywhere-opensource-src-5.7.1
```

Apply [this patch](https://github.com/Homebrew/homebrew-core/issues/3219#issuecomment-235820697):
```shell
curl https://github.com/okeatime/qBittorrent/releases/download/depend.tar.ball/macdeployqt.patch | patch -p1
```

```shell
OPENSSL_LIBS='-L/usr/local/opt/openssl/lib -lssl -lcrypto' ./configure -prefix /usr/local/qt5.7.1 -I/usr/local/opt/openssl/include -no-rpath -opensource -confirm-license -release -openssl-linked -no-securetransport -make libs -make tools -nomake examples -nomake tests -skip qt3d -skip qtactiveqt -skip qtandroidextras -skip qtcanvas3d -skip qtdeclarative -skip qtdoc -skip qtgraphicaleffects -skip qtimageformats -skip qtlocation -skip qtmultimedia -skip qtquickcontrols -skip qtscript -skip qtsensors -skip qtserialport -skip qtwayland -skip qtwebchannel -skip qtwebsockets -skip qtwinextras -skip qtx11extras -skip qtxmlpatterns -skip qtwebview -skip qtwebengine -skip qtconnectivity -v
make -j2 && sudo make install
```

# Download qBittorrent source
Clone from GitHub:
```shell
git clone https://github.com/qbittorrent/qBittorrent.git
```

# Compile qBittorrent
## With Autotools
```shell
export QT_QMAKE=/usr/local/qt5.7.1/bin
./configure
make -j2
$QT_QMAKE/macdeployqt src/qbittorrent.app -dmg
```

## With CMake
```shell
mkdir build && cd build
OPENSSL_ROOT_DIR=/usr/local/opt/openssl Qt5_DIR=/usr/local/qt5.7.1/lib/cmake/Qt5 cmake -DDBUS=OFF ..
make -j2
/usr/local/qt5.7.1/bin/macdeployqt src/app/qbittorrent.app
```

### Optional OpenSSL Linking
```
# use ~/.bashrc if you are using bash
echo export OPENSSL_ROOT_DIR=\"/usr/local/opt/openssl\" >> ~/.zshrc
source ~/.zshrc
```

# Optionally install python for the search function
```shell
# use python3
brew install python3

# or python2
brew install python
```
