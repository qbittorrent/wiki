After stumbling for about 3 hours _(not knowing what the distro package names were to install)_ here is the concise list of commands to build as per:

* https://github.com/qbittorrent/qBittorrent/commit/331219dda828cc2bc7d3c80dc4092401ca5e55d4
* CentOS 7 x64
* 2015-08-09 (Aug)

```
yum -y groupinstall 'Development Tools'
yum -y install qt-devel boost-devel openssl-devel

wget https://github.com/arvidn/libtorrent/releases/download/libtorrent-1_0_6/libtorrent-rasterbar-1.0.6.tar.gz
tar -zxf libtorrent-rasterbar-1.0.6.tar.gz
cd libtorrent-rasterbar-1.0.6
./configure --prefix=/usr
make
make install
ln -s /usr/lib/pkgconfig/libtorrent-rasterbar.pc /usr/lib64/pkgconfig/libtorrent-rasterbar.pc
ln -s /usr/lib/libtorrent-rasterbar.so.8 /usr/lib64/libtorrent-rasterbar.so.8

cd ..
git clone https://github.com/qbittorrent/qBittorrent.git
cd qBittorrent
./configure --prefix=/usr --disable-gui
make
make install
cd

// don't do this if you have a working config file
mkdir -p ~/.config/qBittorrent
cat > ~/.config/qBittorrent/qBittorrent.conf <<- EOM
[Preferences]
WebUI\Enabled=true
EOM
// don't do this if you have a working config file

qbittorrent-nox
```

You're welcome @SoreGums