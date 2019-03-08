[DietPi](https://dietpi.com/ "DietPi Homepage") is a lightweight Debian distro for ARM SoCs such as Raspberry Pi 3 B+ and ASUS Tinker Board. As DietPi is based on Debian 9.0, it ships with a patched version of an older qBittorrent-nox release (3.3.7). qBittorrent 4.x has many improvements to the webUI component which are of particular use for headless operation. 

This guide outlines the steps needed to compile qBittorrent-nox 4.1.x on DietPi 6.x (currently 6.21) and get it to run on boot as a service.  

This guide was made possible by the authors of the [Debian/Ubuntu compilation guide](https://github.com/qbittorrent/qBittorrent/wiki/Compiling-qBittorrent-on-Debian-and-Ubuntu "Debian/Ubuntu compilation guide") and [Guide on running qBittorrent as a service](https://github.com/qbittorrent/qBittorrent/wiki/Setting-up-qBittorrent-on-Ubuntu-server-as-daemon-with-Web-interface-(15.04-and-newer) "Guide on running qBittorrent as a service"). A few DietPi-specific amendments are included. 
    
# Table of Contents

# Table of Contents

* [Dependencies ](#dependencies-)
* [Compiling Libtorrent ](#compiling-libtorrent-)
    * Either
        * [A. git clone from repository](#a-git-clone-from-repository)
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
        * [Create the service file](#create-the-service-file)
        * [Run and check service status](#run-and-check-service-status)
* [Updating qBittorrent-nox ](#updating-qbittorrent-nox-)
    * [Get a copy of the latest qBittorrent release version](#get-a-copy-of-the-latest-qbittorrent-release-version)
    * [check version, install, check version](#check-version-install-check-version)

# Dependencies <a name="dependencies"></a>
You will first need to install various tools and libraries needed for compilation. 

`apt-get install build-essential pkg-config automake libtool git libboost-dev libboost-system-dev libboost-chrono-dev libboost-random-dev libssl-dev libgeoip-dev qtbase5-dev qttools5-dev-tools libqt5svg5-dev zlib1g-dev`

# Compiling Libtorrent <a name="libtorrent"></a>
DietPi's repositories include an older version of [Libtorrent](https://libtorrent.org/ "Libtorrent"). You will need to compile Libtorrent 1.1.x to get qBittorrent-nox 4.x running. 

To get the Libtorrent 1.1.x source code, either

A. `git clone ...`

B. `wget ...`

### A. git clone from repository

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
./configure --disable-debug --enable-encryption --with-libgeoip=system CXXFLAGS=-std=c++11 --with-boost-libdir=/usr/lib/arm-linux-gnueabihf 
make clean
make -j$(nproc)
sudo make install
~~~~~

This will require five to six hours to compile on Raspberry Pi hardware.

## Add Libtorrent as system library <a name="systemlibrary"></a>

You will need to add Libtorrent as a system library or qBittorrent-nox won't run after you compile it.

Create file `/etc/ld.so.conf.d/libtorrent.conf` with contents

~~~~~
/usr/local/lib
~~~~~

Run `sudo ldconfig` afterward.

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
./configure --disable-gui --with-boost-libdir=/usr/lib/arm-linux-gnueabihf
make -j$(nproc)
sudo make install
~~~~~

This will require several hours to compile on Raspberry Pi hardware.

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

Repeat prior steps for downloading the source code and compiling it.

## check version, install, check version

Then stop the service, check the version before install, install, and check the version after install.

~~~~
sudo systemctl stop qbittorrent

/usr/local/bin/qbittorrent-nox --version

sudo make install

/usr/local/bin/qbittorrent-nox --version
~~~~

If the version has changed then the new version was successfully compiled and installed!

Restart the `qbittorrent` service.

~~~~
sudo systemctl start qbittorrent
~~~~