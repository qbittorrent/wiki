After stumbling for about 3 hours _(not knowing what the distro package names were to install)_ here is the concise list of commands to build as per:

* https://github.com/qbittorrent/qBittorrent/commit/452b643e0c449fdd53e159776f0608e7fb2a4014
* CentOS 7 x64
* 2015/04/30

```
yum groupinstall 'Development Tools'
yum install qt-devel
yum install boost-devel
yum install openssl-devel

wget http://sourceforge.net/projects/libtorrent/files/libtorrent/libtorrent-rasterbar-1.0.4.tar.gz
tar -zxf libtorrent-rasterbar-1.0.4.tar.gz
cd libtorrent-rasterbar-1.0.4
./configure --prefix=/usr
make
make install
ln -s /usr/lib/pkgconfig/libtorrent-rasterbar.pc /usr/lib64/pkgconfig/libtorrent-rasterbar.pc
ln -s /usr/lib/libtorrent-rasterbar.so.8 /usr/lib64/libtorrent-rasterbar.so.8

cd..
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

qbittorrent-nox
```

You're welcome @SoreGums