After stumbling for about 2 hours _(getting Qt5 to work)_ here is the how to build qbittorrent as per:

**Update for Qt5**

* https://github.com/qbittorrent/qBittorrent/releases/tag/release-4.0.3
* CentOS 7 x64
* 2018-01-04 (Jan 4th)

Install Dependencies (make sure you have epel repo enabled)
```
yum -y groupinstall "Development Tools"
yum -y install qt-devel boost-devel openssl-devel qt5-qtbase-devel qt5-linguist
```

Grab latest libtorrent release
* libtorrent-1.1.6 at the time of last update

> https://github.com/arvidn/libtorrent/releases check for updates and replace as needed

```
wget https://github.com/arvidn/libtorrent/releases/download/libtorrent-1_1_9/libtorrent-rasterbar-1.1.9.tar.gz
tar -zxf libtorrent-rasterbar-1.1.9.tar.gz
cd libtorrent-rasterbar-1.1.9
./configure --prefix=/usr CXXFLAGS=-std=c++11
make
make install
ln -s /usr/lib/pkgconfig/libtorrent-rasterbar.pc /usr/lib64/pkgconfig/libtorrent-rasterbar.pc
ln -s /usr/lib/libtorrent-rasterbar.so.9 /usr/lib64/libtorrent-rasterbar.so.9
```

Grab latest qbittorrent
```
cd 
git clone https://github.com/qbittorrent/qBittorrent.git (or git pull if already cloned)
cd qBittorrent
./configure --prefix=/usr --disable-gui CPPFLAGS=-I/usr/include/qt5
make
make install
cd
```

***

**Don't do this if you have a working config file ie. you are upgrading**
```
mkdir -p ~/.config/qBittorrent
cat > ~/.config/qBittorrent/qBittorrent.conf <<- EOM
[Preferences]
WebUI\Enabled=true
EOM
```

***

To set up qbittorrent as a deamon see: https://github.com/qbittorrent/qBittorrent/wiki/Setting-up-qBittorrent-as-a-daemon-on-CentOS-7

or else just run `qbittorrent-nox`

>Kudos to @SoreGums

>ref for qmake bug on centos: https://github.com/qbittorrent/qBittorrent/issues/4197#issuecomment-160654335