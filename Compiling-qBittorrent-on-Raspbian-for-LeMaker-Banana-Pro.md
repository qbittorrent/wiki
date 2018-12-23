This how-to will guide you through the **compilation** of **qBittorrent** and **libtorrent-rasterbar** on **LeMaker Banana Pro** with **Raspbian** as Operative System.

This guide is written for Raspbian _(which is based upon Debian)_ and BananaPRO, but the process should be similar for BananaPi and other Debian distributions.

Please continue following this guide only if you have a minimum knowledge of what are you going to do, and only if this is what you really want.

If you are looking only for the latest version of qBittorrent, just head to the download page and use our PPAs.

# Required dependencies

* General required dependencies

   `sudo apt-get install libboost-dev libboost-system-dev`

* Qt4 libraries 

   `sudo apt-get install libqt4-dev`

_[ at the time of this writing, Qt5 libraries were not available for Raspbian on B-Pro. You can safely compile QBitTorrent against Qt4 Libraries. ]_

* Python _(Run time only dependency, needed by the search engine)_ 

   `sudo apt-get install python`

# Optional dependencies

* Geoip Database _(For peer country resolution, strongly advised)_

   ` sudo apt-get install geoip-database`

# Libtorrent

[Libtorrent](https://www.libtorrent.org ) is a library written by Arvid Norberg that qBittorrent depends on. It is necessary to compile and install libtorrent before compiling qBittorrent.

Default Raspbian distro packages a very old _(and very unstable)_ version of  libtorrent, hence you will need to compile it yourself.

### Notes:

The procedure for compiling and installing Libtorrent on B-Pro is derived from the [Deluge Wiki](https://dev.deluge-torrent.org/wiki/Building/libtorrent): some minor adjustments have been made to the procedure in order to work correctly on B-Pro.

## Install dependencies for libtorrent build automatically, using build-dep:
    sudo apt-get build-dep libtorrent-rasterbar
    sudo apt-get install checkinstall

   Your system could be configured with some option that might interfere with checkinstall.
   If previous installation fails then procede manually:

    sudo apt-get install build-essential checkinstall libboost-system-dev libboost-python-dev libssl-dev libgeoip-dev

## Download ​libtorrent and extract:
`   wget https://github.com/arvidn/libtorrent/releases/download/libtorrent_1_1_11/libtorrent-rasterbar-1.1.11.tar.gz`

   `tar xf libtorrent-rasterbar-1.0.10.tar.gz`

   `cd libtorrent-rasterbar-1.0.10`

## Configure:

   `./configure --enable-python-binding --with-libgeoip --with-libiconv --with-qt4`

## WARNING: Users with Debian 8 "Jessie" or "Stretch"
If you have updated your Raspbian to the latest version "Jessie" then you will most certainly face an error with _libboost library_ while using the ._/configure_

To overcome this error and configure correctly you must issue the following commands:

    sudo apt-get install libboost-chrono-dev libboost-random-dev

    ./configure --enable-python-binding --with-libgeoip --with-libiconv --with-boost-libdir=/usr/lib/arm-linux-gnueabihf

## Build:

   `make -j$(nproc)`

   _[The make option -j$(nproc) will utilize all available cpu cores.]_

## Install library and python bindings:

   `sudo checkinstall`
   
   _[ The `checkinstall `command replaces the `make install`. It created a DEB Package for easier removal/re-install trough dpkg. ]_

   `sudo ldconfig` 

   _[Running `ldconfig` avoids an ImportError for libtorrent-rasterbar.so, a result of Python being unable to find the main library.]_

# Compiling qBittorrent (with the GUI)

Now that each prerequisite has been completed you mut obtain the qBittorrent source code. 

Either download and extract a .tar archive from [Sourceforge]( https://sourceforge.net/projects/qbittorrent/files/qbittorrent/) or use the following commands to speed-up the process.

`   git clone https://github.com/qbittorrent/qBittorrent`

## Configure qBittorrent

## WARNING: Users with Debian 8 "Jessie"
If you have updated your Raspbian to the latest version "Jessie" then you will most certainly face an error with _libboost library_ while using the ._/configure_

To overcome this error and configure correctly you must issue the following commands:

    sudo apt-get install libboost-dev-all

    ./configure --enable-python-binding --with-libgeoip --with-libiconv --with-boost-libdir=/usr/lib/arm-linux-gnueabihf --with-qt4

   `cd qBittorrent/`

   `./configure --prefix=/usr --with-qt4`

   `make`

## Install qBittorrent:

   `sudo checkinstall`

   _[alternatively you can use the standard command `make install`, as discussed before]_

**That's it! qBittorrent should now be installed.** You can now run qBittorrent using the following command:

   `qbittorrent`

# Compiling qBittorrent-NOX _(aka Without the GUI, also called headless)_


Now that each prerequisite has been completed you mut obtain the qBittorrent source code. 

Either download and extract a .tar archive from [Sourceforge]( https://sourceforge.net/projects/qbittorrent/files/qbittorrent/) or use the following commands to speed-up the process.

`   git clone https://github.com/qbittorrent/qBittorrent`

## Configure qBittorrent

   `cd qBittorrent/`

   `./configure --prefix=/usr --disable-gui --with-qt4`

## WARNING: Users with Debian 8 "Jessie"
If you have updated your Raspbian to the latest version "Jessie" then you will most certainly face an error with _libboost library_ while using the ._/configure_

To overcome this error and configure correctly you must issue the following commands:

    sudo apt-get install libboost-all-dev

    ./configure --prefix=/usr --disable-gui --enable-python-binding --with-libgeoip --with-libiconv --with-boost-libdir=/usr/lib/arm-linux-gnueabihf --with-qt4

   `make`

## Install qBittorrent:

   `sudo checkinstall`

   _[alternatively you can use the standard command `make install`, as discussed before]_

**That's it! qBittorrent should now be installed.** You could simply launch Qbittorrent-nox using the following comand:

   'qbittorrent-nox'

But since you have chosen the "headless" version you can access qBittorrent only through its WEB Interface.

### Standard Access to WEB Interface

As a default, you can access it from the following address:

 _http://localhost:8080_

Using the following credentials:

 _Username: admin_

 _Password: adminadmin_

### Customize WEB Interface Listening Port

If you have another service listening on port 8080, or you simply want to use a different port, you can instruct qBittorrent to listen on a different by using the following launch command:

   `qbittorrent-nox --webui-port=xxxx`

Where the 'xxxx' stands for the port number you desire to use. For example, if you want to use the Port 8181 than the command will be:

   `qbittorrent-nox --webui-port=8181`

**Note:** There is no need to repeat this instruction in future. Once started for the first time, qBittorrent will remember the designated listening port. So from next sessions onwards you can simply execute it with the command:

   `qbittorrent-nox`

# Running qBittorrent-nox as a daemon at system startup

Since you are using the "headless" version of qBittorrent, you might also want to autostart it at system boot as a _daemon_.

By doing so you will not need to keep open a terminal window in order to execute qBittorrent.

## Using CronJob at Reboot

_**Please Note:**_ **the following solution is based upon personal research** and days of testing. I am aware that it might not be the nicest solution available, but if your are **running qBittorrent-nox on B-Pro with Raspbian** chances are that you too noticed how **impredictible **and **unstable **is its behaviour when using **`update-rc.d` method.**

_The following commands are based upon the assumption that you are running qBittorrent under a dedicated user._ 
If you are running it under the 'bananapi' root user you need to execute the following with 'root privileges'.

Exceute the following command:

   `crontab -e`

Scroll to the bottom of the file and insert the following line:

   `@reboot qbittorrent-nox`

Use " CTRL + O " to save the file, than " CTRL + X " to exit.

That's it! **Now qBittorrent-nox should be correctly configured for autostart!** 

If you want to try it just reboot you B-Pro with the command:

   `sudo reboot`

## Final Notes

This work has been realized putting togheter personal knowledge and information provided by other authors.

**Credits are as follows:**

**Thanks to** [**SledgeHammer**](https://github.com/qbittorrent/qBittorrent/wiki/Compiling-qBittorrent-on-Debian-and-Ubuntu) for providing a solid template for compiling and running qBittorrent under Debian. 

**Thanks to** [**Cas**](https://dev.deluge-torrent.org/wiki/Building/libtorrent) from Deluge Wiki for explaining how to compile libtorrent_rastebar under Debian with the use of checkinstall.
