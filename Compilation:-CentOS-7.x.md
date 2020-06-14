This how-to will guide you though the compilation of qBittorrent and
`libtorrent-rasterbar`.

This guide is written for CentOS 7.x, but the process should be similar
for other RHEL distributions.

# Required dependencies

## General required dependencies

```bash
sudo yum groupinstall "Development Tools"
sudo yum install devtoolset-8-gcc devtoolset-8-gcc-c++
sudo yum install qt-devel openssl-devel qt5-qtbase-devel qt5-linguist
```

## Boost

[Download](https://www.boost.org/users/download/) latest version of
Boost. (Actually 1.72.0)

`wget https://dl.bintray.com/boostorg/release/1.72.0/source/boost_1_72_0.tar.gz`

## Qt libraries

qBittorrent 4.0 - 4.1.x requires at least Qt 5.5.1, and qBittorrent 4.2
and later requires at least Qt 5.9.

Check installed version:

```bash
$ rpm -qa | grep qt5-qtbase
qt5-qtbase-common-5.9.7-2.el7.noarch
qt5-qtbase-gui-5.9.7-2.el7.x86_64
qt5-qtbase-5.9.7-2.el7.x86_64
qt5-qtbase-devel-5.9.7-2.el7.x86_64
```

## Libtorrent

[Libtorrent](https://libtorrent.org/) is a library written by Arvid Norberg that qBittorrent depends on.
It is necessary to compile and install `libtorrent` before compiling qBittorrent.

### Boost

Change devtoolsset if you didn't do that already: `scl enable devtoolset-8 bash`

Compile:

```bash
export DIR_BOOST="/opt/boost"
tar -xvf boost_1_72_0.tar.gz
cd boost_1_72_0
./bootstrap.sh --prefix=${DIR_BOOST}
./b2 install --prefix=${DIR_BOOST} --with=all -j$(( $(nproc) - 1 ))
```

### Libtorrent

Change devtoolsset if you didn't do that already: `scl enable devtoolset-8 bash`

Clone from the repository: `git clone --depth 1 -b RC_1_2 https://github.com/arvidn/libtorrent.git`

If you need to build deluge,you must run the following command:
```bash
yum -y install python-devel boost-devel
```
Compile:

```bash
cd libtorrent
./autotool.sh

# Only compiled for qbittorrent:
./configure --prefix=/usr --disable-debug --enable-encryption --with-boost=${DIR_BOOST} CXXFLAGS=--std=c++14

# Compiling for both qbittorrent and deluge:
./configure --prefix=/usr --disable-debug --enable-encryption --with-boost=${DIR_BOOST} CXXFLAGS=--std=c++14 --with-libiconv --with-libgeoip=system --enable-python-binding
make -j$(( $(nproc) - 1 ))
make install
ln -s /usr/lib/pkgconfig/libtorrent-rasterbar.pc /usr/lib64/pkgconfig/libtorrent-rasterbar.pc
```

**Last command was missing and on 64bit systems will fail without it.Here is the error information:**
```txt
checking for libtorrent... no
configure: error: Package requirements (libtorrent-rasterbar >= 1.0.6) were not met:

No package 'libtorrent-rasterbar' found

Consider adjusting the PKG_CONFIG_PATH environment variable if you
installed software in a non-standard prefix.

Alternatively, you may set the environment variables libtorrent_CFLAGS
and libtorrent_LIBS to avoid the need to call pkg-config.
See the pkg-config man page for more details.
```

# Compiling qBittorrent (without the GUI)

First, obtain the qBittorrent source code.

Either download and extract a `.tar` archive from [the GitHub releases page](https://github.com/qbittorrent/qBittorrent/releases) or clone the git repository: `git clone --depth 1 -b v4_2_x https://github.com/qbittorrent/qBittorrent`

Change devtoolsset if you didn't do that already: `scl enable devtoolset-8 bash`

Compile:

```bash
cd qBittorrent
./configure --prefix=/usr --disable-gui CPPFLAGS=-I/usr/include/qt5 --with-boost=${DIR_BOOST} CXXFLAGS=--std=c++14
make -j$(( $(nproc) - 1 ))
make install
```

Since you disabled the graphical user interface, qBittorrent can only be controlled via its WebUI.
By default, you can access it at `http://localhost:8080` with the default credentials:

```txt
Username: admin
Password: adminadmin
```

Documentation about running qBittorrent without GUI is available [here](https://github.com/qbittorrent/qBittorrent/wiki/Running-qBittorrent-without-X-server-(WebUI-only)).

To set up qbittorrent as a deamon see [this guide](https://github.com/qbittorrent/qBittorrent/wiki/Setting-up-qBittorrent-as-a-daemon-on-CentOS-7)

# Troubleshooting

If are you facing a problem like this:

```txt
qbittorrent-nox: error while loading shared libraries: libtorrent-rasterbar.so 10: cannot open shared object file: No such file or directory
```


This often happened when you are using 64-bit CentOS 7.x.
And it's because of the libraries that the qBittorrent need are not in `/usr/lib64/`.

You can simply create a soft link to solve it. Do it like this:

```bash
ln -s /usr/lib/libtorrent-rasterbar.so.10 /usr/lib64/libtorrent-rasterbar.so.10
```

For missing `libboost_system.so.1.72.0`:

```bash
ln -s /opt/boost/lib/libboost_system.so.1.72.0 /usr/lib64/libboost_system.so.1.72.0
```
