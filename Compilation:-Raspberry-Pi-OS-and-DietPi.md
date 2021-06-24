[Raspberry Pi OS](https://www.raspberrypi.org/software/) is the most popular Linux distribution built for Raspberry Pi hardware.

[DietPi](https://dietpi.com/ "DietPi Homepage") is a refined Linux distribution for ARM SoCs such as Raspberry Pi 3 B+ and ASUS Tinker Board.

Both are based on Debian. Debian 9.0 ships with a patched version of an older qBittorrent-nox release (4.1.5). qBittorrent 4.3.x has many improvements to the webUI component which are of particular use for headless operation for example RSS handling.

This guide outlines the steps needed to compile qBittorrent-nox 4.3.x and run it as a service. Both DietPi and Raspberry Pi OS provide pre-compiled qBittorrent using either `dietpi-software` or `apt`. Use this guide if you want to run the most recent qBittorrent (and libtorrent-rasterbar).

This guide was made possible by the authors of the [Debian/Ubuntu compilation guide](https://github.com/qbittorrent/qBittorrent/wiki/Compilation:-Debian-and-Ubuntu "Debian/Ubuntu compilation guide") and [Guide on running qBittorrent as a service](https://github.com/qbittorrent/qBittorrent/wiki/Setting-up-qBittorrent-on-Ubuntu-server-as-daemon-with-Web-interface-(15.04-and-newer) "Guide on running qBittorrent as a service"). A few DietPi-specific amendments are included.

# Table of Contents

* [Dependencies](#dependencies)
* [Compiling Libtorrent](#compiling-libtorrent)
	* [Get the source code](#get-the-source-code)
        * [A. `git clone` from repository](#a-git-clone-from-repository)
        * [B. Download the latest release](#b-download-the-latest-release)
    * [Compile Libtorrent](#compile-libtorrent)
    * [Add Libtorrent as a system library](#add-libtorrent-as-a-system-library)
* [Compiling qBittorrent-nox](#compiling-qbittorrent-nox)
	* [Get the source code](#get-the-source-code-1)
        * [A. `git clone` from repository](#a-git-clone-from-repository-1)
        * [B. Download the latest release](#b-download-the-latest-release-1)
    * [Compile qBittorrent-nox](#compile-qbittorrent-nox)
* [Running qBittorrent-nox on boot](#running-qbittorrent-nox-on-boot)
    * [Add user for qBittorrent-nox service](#add-user-for-qbittorrent-nox-service)
    * [Create systemd service file](#create-systemd-service-file)
    * [Run and check systemd service status](#run-and-check-systemd-service-status)
* [Updating qBittorrent-nox](#updating-qbittorrent-nox)
    * [Updating an already compiled version of qBittorrent-nox](#updating-an-already-compiled-version-of-qBittorrent-nox)
    * [Checking the version to verify that the binary got updated](#checking-the-version-to-verify-that-the-binary-got-updated)

# Dependencies

You will first need to install various tools and libraries needed for compilation.

~~~~
sudo apt install build-essential pkg-config automake libtool libc6-dev libboost-dev libboost-system-dev libboost-chrono-dev libboost-random-dev libssl-dev qtbase5-dev qttools5-dev-tools libqt5svg5-dev zlib1g-dev
~~~~

If you choose to retrieve source code using `git clone`, then also `sudo apt install git`.

# Compiling Libtorrent

DietPi's and Raspberry Pi OS's repositories include older versions of [Libtorrent](https://libtorrent.org/). You will need to compile Libtorrent 1.2.x to get qBittorrent-nox 4.2.x running. Both methods below outline how to install Libtorrent 1.2.x for use with qBittorrent-nox 4.3.x (and likely later versions).

## Get the source code

Two versions of Libtorrent are currently maintained: 1.2.x and 2.0.x.

Since release 4.2.0, qBittorrent makes use of the 1.2.x version, so get the 1.2.x release.

To get the Libtorrent source code, either:

A. `git clone ...`

B. `wget ...`

### A. `git clone` from repository

~~~~
git clone https://github.com/arvidn/libtorrent.git
cd libtorrent
# select the latest release tag
git checkout $(git tag | grep v1\.2.\. | sort -t _ -n -k 3 | tail -n 1)
~~~~

### B. Download the [latest release](https://github.com/arvidn/libtorrent/releases)

~~~~
wget https://github.com/arvidn/libtorrent/archive/libtorrent-1_2_14.zip
unzip libtorrent-1_2_14.zip
cd libtorrent-libtorrent-1_2_14
~~~~

## Compile Libtorrent

~~~~
./autotool.sh
./configure --with-boost-libdir=/usr/lib/arm-linux-gnueabihf --with-libiconv CXXFLAGS="-std=c++17"
make -j$(nproc)
sudo make install
~~~~

#### out of memory (OOM)

If OOM errors occur, add a swap file.

~~~~
sudo dd if=/dev/zero of=/.swapfile bs=1M count=1024
sudo mkswap /.swapfile
sudo swapon /.swapfile
sudo swapon -s  # check swap is activated
make
# assuming the prior command succeeded
sudo swapoff /.swapfile
sudo rm /.swapfile
~~~~

(Those commands were copied from [here](https://dev.deluge-torrent.org/wiki/Building/libtorrent#TemporarySwapFileforRasperryPiorlowmemorysystems).)

One example manifestation of an OOM error on Raspberry Pi OS looks like:

~~~~
$ make
...
make[1]: Entering directory '/tmp/libtorrent-libtorrent_1_2_0/src'
  CXX      libtorrent_rasterbar_la-session_impl.lo
g++: internal compiler error: Killed (program cc1plus)
~~~~

## Add Libtorrent as a system library

You will need to add Libtorrent as a system library or qBittorrent-nox won't run after you compile it.

Create a file `sudo nano /etc/ld.so.conf.d/libtorrent.conf` with contents `/usr/local/lib`.

Run `sudo ldconfig` afterward.

Check if your `LD_LIBRARY_PATH` environment variable is set and the path `/usr/local/lib` is included: run `env` in your terminal and look for `LD_LIBRARY_PATH`.

If so, you are good to go. If not, add the path to the variable: `export LD_LIBRARY_PATH=/usr/local/lib:${LD_LIBRARY_PATH}`

#  Compiling qBittorrent-nox

## Get the source code

To get the qBittorrent-nox source code, either:

A. `git clone ...`

B. `wget ...`

### A. `git clone` from repository

~~~~
git clone -b v4_3_x https://github.com/qbittorrent/qBittorrent
cd qBittorrent
~~~~

You may select the branch version on the [branches page](https://github.com/qbittorrent/qBittorrent/branches).

### B. Download the [latest release](https://github.com/qbittorrent/qBittorrent/releases)

~~~~
wget https://github.com/qbittorrent/qBittorrent/archive/release-4.3.5.zip
unzip release-4.3.5.zip
cd qBittorrent-release-4.3.5
~~~~

## Compile qBittorrent-nox

~~~~
./configure --disable-gui --enable-systemd --with-boost-libdir=/usr/lib/arm-linux-gnueabihf CXXFLAGS="-std=c++17"
make -j$(nproc)
sudo make install
~~~~

NOTE: Review [Ubuntu/Debian compilation guide](https://github.com/qbittorrent/qBittorrent/wiki/Compiling-qBittorrent-on-Debian-and-Ubuntu#Compiling_qBittorrent_with_the_GUI) if you want to run qBittorrent with a GUI.

The binary should be located at `/usr/local/bin/qbittorrent-nox`. If `qbittorrent-nox` was installed using `apt` then that binary will be at `/usr/bin/qbittorrent-nox`. _Do not confuse them!_

**Web UI access informations**
* _http://localhost:8080_
* Username: _admin_
* Password: _adminadmin_

qBittorrent-nox is currently installed as a terminal application, which is not optimal for headless use. We now will add qBittorrent-nox as a service.

# Running qBittorrent-nox on boot

## Add user for qBittorrent-nox service

For Raspberry Pi OS:
~~~~
sudo useradd -rm qbittorrent -G pi -s /usr/sbin/nologin
~~~~

For DietPi:
~~~~
sudo useradd -rm qbittorrent -G dietpi -s /usr/sbin/nologin
~~~~

## Create systemd service file

_UPDATE:_ this **may** not be necessary if qBittorrent compilation was configured with flag `--enable-systemd`.

Create a systemd service file `sudo nano /etc/systemd/system/qbittorrent.service`.

Contents for Raspberry Pi OS:
~~~~
Description=qBittorrent Daemon Service
After=network.target

[Service]
User=qbittorrent
Group=pi
ExecStart=/usr/local/bin/qbittorrent-nox
ExecStop=/usr/bin/killall -w qbittorrent-nox

[Install]
WantedBy=multi-user.target
~~~~

Contents for DietPi:
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
~~~~

## Run and check systemd service status

~~~~
sudo systemctl daemon-reload
sudo systemctl start qbittorrent
sudo systemctl status qbittorrent
~~~~

The `systemctl status` command should show qBittorrent-nox is `active (running)`.

Enable `qbittorrent` service during boot: `sudo systemctl enable qbittorrent`.

# Updating qBittorrent-nox

## Updating an already compiled version of qBittorrent-nox

Get the latest version of qBittorrent using instructions outlined above (git/wget).

Navigate to the qBittorrent directory with the latest release.

~~~~
sudo systemctl stop qbittorrent
./configure --disable-gui --enable-systemd --with-boost-libdir=/usr/lib/arm-linux-gnueabihf CXXFLAGS="-std=c++17"
make -j$(nproc)
sudo make install
~~~~

## Checking the version to verify that the binary got updated

~~~~
sudo systemctl stop qbittorrent
/usr/local/bin/qbittorrent-nox --version
~~~~

If the version has changed then the new version was successfully compiled and installed!

Restart the `qbittorrent` service: `sudo systemctl start qbittorrent`.