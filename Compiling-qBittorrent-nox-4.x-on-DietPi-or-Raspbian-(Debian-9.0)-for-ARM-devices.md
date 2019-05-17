[Raspbian](https://www.raspbian.org/) is the most popular Linux distribution built for Raspberry Pi hardware.

[DietPi](https://dietpi.com/ "DietPi Homepage") is a refined Linux distribution for ARM SoCs such as Raspberry Pi 3 B+ and ASUS Tinker Board.

Both are based on Debian.  Debian 9.0 ships with a patched version of an older qBittorrent-nox release (3.3.7). qBittorrent 4.x has many improvements to the webUI component which are of particular use for headless operation.

This guide outlines the steps needed to compile qBittorrent-nox 4.1.x and run it as a service.  Both DietPi and Raspbian provide pre-compiled qBittorrent using either `dietpi-software` or `apt`.  Use this guide if you want to run the most recent qBittorrent (and libtorrent-rasterbar).

This guide was made possible by the authors of the [Debian/Ubuntu compilation guide](https://github.com/qbittorrent/qBittorrent/wiki/Compiling-qBittorrent-on-Debian-and-Ubuntu "Debian/Ubuntu compilation guide") and [Guide on running qBittorrent as a service](https://github.com/qbittorrent/qBittorrent/wiki/Setting-up-qBittorrent-on-Ubuntu-server-as-daemon-with-Web-interface-(15.04-and-newer) "Guide on running qBittorrent as a service"). A few DietPi-specific amendments are included. 

# Table of Contents

* [Dependencies ](#dependencies-)
* [Compiling Libtorrent ](#compiling-libtorrent-)
    * Either
        * [A. `git clone` from repository](#a-git-clone-from-repository)
        * [B. download the latest release](https://github.com/arvidn/libtorrent/releases)
    * [Compile Libtorrent 1.1.x](#compile-libtorrent-11x)
    * [Add Libtorrent as system library ](#add-libtorrent-as-system-library-)
* [Compiling qBittorrent-nox ](#compiling-qbittorrent-nox-)
    * [Compile 4.1.x version](#compile-41x-version)
        * [A. `git clone` from repository](#a-git-clone-from-repository-1)
        * [B. Download the release source code](https://github.com/qbittorrent/qBittorrent/releases)
    * [Compile qBittorrent-nox](#compile-qbittorrent-nox)
* [Running qBittorrent-nox on boot ](#running-qbittorrent-nox-on-boot-)
    * [Add user for qBittorrent-nox service](#add-user-for-qbittorrent-nox-service)
    * [Create systemd service file](#create-systemd-service-file)
    * [Run and check systemd service status](#run-and-check-systemd-service-status)
* [Updating qBittorrent-nox ](#updating-qbittorrent-nox-)
    * [Get a copy of the latest qBittorrent release version](#get-a-copy-of-the-latest-qbittorrent-release-version)
    * [check version, install, check version](#check-version-install-check-version)

# Dependencies <a name="dependencies"></a>
You will first need to install various tools and libraries needed for compilation. 

~~~~
sudo apt-get install \
    build-essential \
    pkg-config \
    automake \
    libtool \
    libc6-dev \
    libboost-dev \
    libboost-system-dev \
    libboost-chrono-dev \
    libboost-random-dev \
    libssl-dev \
    qtbase5-dev \
    qttools5-dev-tools \
    libqt5svg5-dev \
    zlib1g-dev
~~~~

If you choose to retrieve source code using `git clone` then also `sudo apt-get install git`.

# Compiling Libtorrent <a name="libtorrent"></a>
DietPi's repositories include an older version of [Libtorrent](https://libtorrent.org/ "Libtorrent"). You will need to compile Libtorrent 1.1.x to get qBittorrent-nox 4.x running. 

To get the Libtorrent 1.1.x source code, either

A. `git clone ...`

B. `wget ...`

### A. `git clone` from repository

~~~~
git clone https://github.com/arvidn/libtorrent.git
cd libtorrent
# select the latest release tag
git checkout $(git tag | grep libtorrent-1_1_ | sort -t _ -n -k 3 | tail -n 1)
~~~~~

### B. download [the latest release](https://github.com/arvidn/libtorrent/releases)

using release *libtorrent_1_2_0* in this example
~~~~~
wget https://github.com/arvidn/libtorrent/archive/libtorrent_1_2_0.zip
unzip libtorrent_1_2_0.zip
cd libtorrent-libtorrent_1_2_0
~~~~~


## Compile Libtorrent 1.1.x

~~~~
./autotool.sh
export CXXFLAGS=-std=c++11
./configure \
    --disable-debug \
    --enable-encryption \
    --with-boost-libdir=/usr/lib/arm-linux-gnueabihf \
    --with-libiconv
make -j$(nproc)
sudo make install
~~~~~

#### out of memory (OOM)

If OOM errors occur then add a swap file.

~~~~~
sudo dd if=/dev/zero of=/.swapfile bs=1M count=1024
sudo mkswap /.swapfile
sudo swapon /.swapfile
sudo swapon -s  # check swap is activated
make
# assuming the prior command succeeded
sudo swapoff /.swapfile
sudo rm /.swapfile
~~~~~

(Those commands were copied from [here](https://dev.deluge-torrent.org/wiki/Building/libtorrent#TemporarySwapFileforRasperryPiorlowmemorysystems)).

One example manifestation of an OOM error on Raspbian OS looks like

~~~~~
$ make
...
make[1]: Entering directory '/tmp/libtorrent-libtorrent_1_2_0/src'
  CXX      libtorrent_rasterbar_la-session_impl.lo
g++: internal compiler error: Killed (program cc1plus)
~~~~~

## Add Libtorrent as system library <a name="systemlibrary"></a>

You will need to add Libtorrent as a system library or qBittorrent-nox won't run after you compile it.

Create file `/etc/ld.so.conf.d/libtorrent.conf` with contents

~~~~~
/usr/local/lib
~~~~~

Run `sudo ldconfig` afterward.

Check if your <code>LD_LIBRARY_PATH</code> environment variable is set and the path <code>/usr/local/lib</code> is included.

Simply run <code>env</code> in your terminal and look for <code>LD_LIBRARY_PATH</code>.
If so, you are good to go. If not, add the path to the variable:
  export LD_LIBRARY_PATH=/usr/local/lib:${LD_LIBRARY_PATH}

#  Compiling qBittorrent-nox <a name="qbittorrentnox"></a>

## Compile 4.1.x version

To get the qBittorrent-nox source code, either

A. compile a cloned git repository _or_

B. download a release source code.

### A. `git clone` from repository

~~~~
git clone -b v4_1_x https://github.com/qbittorrent/qBittorrent
cd qBittorrent
~~~~~

You may select the branch version on the [branches page](https://github.com/qbittorrent/qBittorrent/branches). 

### B. Download the [release source code](https://github.com/qbittorrent/qBittorrent/releases)

Using release *release-4.1.5* in this example,

~~~~
wget https://github.com/qbittorrent/qBittorrent/archive/release-4.1.5.zip
unzip release-4.1.5.zip
cd qBittorrent-release-4.1.5
~~~~~

## Compile qBittorrent-nox

~~~~
./configure --disable-gui --enable-systemd --with-boost-libdir=/usr/lib/arm-linux-gnueabihf
make -j$(nproc)
sudo make install
~~~~~

NOTE: Review [Ubuntu/Debian compilation guide](https://github.com/qbittorrent/qBittorrent/wiki/Compiling-qBittorrent-on-Debian-and-Ubuntu#Compiling_qBittorrent_with_the_GUI) if you want to run qBittorrent with a GUI.

The binary should be located at `/usr/local/bin/qbittorrent-nox`.  If `qbittorrent-nox` was installed using `apt` then that binary will be at `/usr/bin/qbittorrent-nox`.  _Do not confuse them!_

**WebUI access information**
* _http://localhost:8080_
* Username: _admin_
* Password: _adminadmin_

qBittorrent-nox is currently installed as a terminal application, which is not optimal for headless use. We now will add qBittorrent-nox as a service.

# Running qBittorrent-nox on boot <a name="onboot"></a>

## Add user for qBittorrent-nox service

~~~~
sudo useradd -rm qbittorrent -G dietpi -s /usr/sbin/nologin
~~~~

## Create systemd service file

_UPDATE:_ this may not be necessary if qBittorrent compilation was configured with flag `--enable-systemd`.

Create a systemd service file at `/etc/systemd/system/qbittorrent.service`.  Contents are:

~~~~
Description=qBittorrent Daemon Service
After=network.target

[Service]
User=qbittorrent
Group=dietpi
ExecStart=/usr/local/bin/qbittorrent-nox
ExecStop=/usr/bin/killall -w qbittorrent-nox

[Install]
WantedBy=multi-user.target
~~~~~   

## Run and check systemd service status
~~~~
sudo systemctl daemon-reload
sudo systemctl start qbittorrent
sudo systemctl status qbittorrent
~~~~~

The `systemctl status` command should show qBittorrent-nox is active and running.

Enable `qbittorrent` service during boot.

~~~~
sudo systemctl enable qbittorrent
~~~~~

# Updating qBittorrent-nox <a name="upqbt"></a>

## Get a copy of the latest qBittorrent release version

On DietPi, you will need to run the following commands to update an already compiled version of qBittorrent-nox:
~~~~
systemctl stop qbittorrent
./configure --disable-gui --with-boost-libdir=/usr/lib/arm-linux-gnueabihf --prefix=/usr/local/bin/
sudo make -j$(nproc)
make install
~~~~

## check version to verify that the binary got updated. 

~~~~
sudo systemctl stop qbittorrent

/usr/local/bin/qbittorrent-nox --version
~~~~

If the version has changed then the new version was successfully compiled and installed!

Restart the `qbittorrent` service.

~~~~
sudo systemctl start qbittorrent
~~~~