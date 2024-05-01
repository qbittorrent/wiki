This how-to will guide you though the compilation of qBittorrent and libtorrent-rasterbar.<br />
This guide is written for CentOS 8.x, but the process should be similar for other RHEL distributions.

First of all, you need install

```bash
yum install "https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm"
```

***

## General required dependencies

### Running the binary 

```bash
# step 1

yum install -y autoconf automake boost-devel boost-system gcc gcc-c++ git glib2 glibc gmp gnutls libblkid libcap libffi libgcc libgcrypt libgpg-error libicu libidn2 libmount libselinux libstdc++ libtasn1 libtool libunistring libuuid lz4-libs make nettle openssl-devel openssl-libs p11-kit pcre pcre2 qt5-qtbase systemd-libs tar wget xz-libs zlib

# step 2 (install libtorrent)

git clone -b RC_1_XXXXXXXXXXX https://github.com/arvidn/libtorrent/
cd libtorrent
./autotool.sh
./configure
make -j 
make install
```

### compile need

```bash
# step 4

yum install -y qt5-linguist qt5-qttools-devel qt5-qtsvg-devel

# step 5

git clone -b v4_2_XXXXXXXXXXX https://github.com/qbittorrent/qBittorrent
cd qBittorrent
./configure --prefix=/usr --disable-gui
make -j
make install
```
