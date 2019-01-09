[DietPi](https://dietpi.com/ "DietPi Homepage") is a lightweight Debian distro for ARM SoCs such as Raspberry Pi 3 B+ and ASUS Tinker Board. As DietPi is based on Debian 9.0, it ships with a patched version of an older qBittorrent-nox release (3.3.7). qBittorrent 4.x has many improvements to the webUI component which are of particular use for headless operation. 

This guide outlines the steps needed to compile qBittorrent-nox 4.1.x on DietPi 6.16 and get it to run on boot as a service.  

This guide was made possible by the authors of the [Debian/Ubuntu compilation guide](https://github.com/qbittorrent/qBittorrent/wiki/Compiling-qBittorrent-on-Debian-and-Ubuntu "Debian/Ubuntu compilation guide") and [Guide on running qBittorrent as a service](https://github.com/qbittorrent/qBittorrent/wiki/Setting-up-qBittorrent-on-Ubuntu-server-as-daemon-with-Web-interface-(15.04-and-newer) "Guide on running qBittorrent as a service"). A few DietPi-specific amendments are included. 
    
# Table of contents
1. [Dependencies](#dependencies)
2. [Compiling Libtorrent](#libtorrent)
    1. [Add Libtorrent as system library](#systemlibrary)
3. [Compiling qBittorrent-nox](#qbittorrentnox)
4. [Running qBittorrent-nox on boot](#onboot)

## Dependencies <a name="dependencies"></a>
You will first need to install various tools and libraries needed for compilation. 

`apt-get install build-essential pkg-config automake libtool git libboost-dev libboost-system-dev libboost-chrono-dev libboost-random-dev libssl-dev libgeoip-dev qtbase5-dev qttools5-dev-tools libqt5svg5-dev zlib1g-dev`

## Compiling Libtorrent <a name="libtorrent"></a>
DietPi's repositories include an older version of [Libtorrent](https://dietpi.com/ "Libtorrent"). You will need to compile Libtorrent 1.1.x to get qBittorrent-nox 4.x running. 

**Clone from repo**
~~~~
git clone https://github.com/arvidn/libtorrent.git
cd libtorrent
~~~~~   
**Select Libtorrent 1.1.x**
~~~~
git checkout $(git tag | grep libtorrent-1_1_ | sort -t _ -n -k 3 | tail -n 1)
./autotool.sh
./configure --disable-debug --enable-encryption --with-libgeoip=system CXXFLAGS=-std=c++11 --with-boost-libdir=/usr/lib/arm-linux-gnueabihf 
~~~~~ 
**Start Compilation**
~~~~
make clean && make -j$(nproc)
make install
~~~~~ 

### Add Libtorrent as system library <a name="systemlibrary"></a>
You will need to add Libtorrent as a system library or qBittorrent-nox won't run after you compile it.
 
**Add Libtorrent as a system library** 
~~~~
cd /etc/ld.so.conf.d/
nano libtorrent.conf
~~~~~
**Add Libttorrent's location** 
~~~~
/usr/local/lib
~~~~~
That path above should be the only thing in the libtorrent.conf file. Run the `ldconfig` command after exiting nano. Now proceed to compiling qBittorrent-nox.  

##  Compiling qBittorrent-nox <a name="qbittorrentnox"></a>

**Clone 4.1.x branch**
~~~~
git clone -b v4_1_x https://github.com/qbittorrent/qBittorrent
cd qBittorrent
~~~~~
NOTE: You can select the branch version on [this](https://github.com/qbittorrent/qBittorrent/branches) page. 

**Compile qBittorrent-nox**
~~~~
./configure --disable-gui --with-boost-libdir=/usr/lib/arm-linux-gnueabihf
make -j$(nproc) 
make install
~~~~~
NOTE: Consult the [Ubuntu/Debian compilation guide](https://github.com/qbittorrent/qBittorrent/wiki/Compiling-qBittorrent-on-Debian-and-Ubuntu#Compiling_qBittorrent_with_the_GUI) if you want to run qBittorrent with a GUI.

You can run qBittorrent-nox using the `qbittorrent-nox` command. The binary is located in `/usr/local/bin/`. 

**WebUI access information**
* http://localhost:8080
* Username: admin
* Password: adminadmin

qBittorrent-nox is currently installed as a terminal application, which is not optimal for headless use. We now will add qBittorrent-nox as a service.     

##  Running qBittorrent-nox on boot <a name="onboot"></a>          
**Add user for qBittorrent-nox service**
~~~~
useradd -rm qbittorrent -G dietpi -s /usr/sbin/nologin
~~~~
**Create the service file**
~~~~
cd /etc/systemd/system/
nano qbittorrent.service
~~~~~
**Add service template** 
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
**Run and check service status**
~~~~
systemctl daemon-reload
systemctl start qbittorrent
systemctl status qbittorrent
~~~~~
When you run the status command, you should get a response that qBittorrent-nox is active and running. 

Finally, enable starting the service on boot. 
~~~~
systemctl enable qbittorrent
~~~~~