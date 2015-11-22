qBittorrent has a feature-rich Web UI allowing users to control qBittorrent remotely. This is ideal for headless servers without the X window system such as Ubuntu Server.

Note, this was tested on Ubuntu Server 12.04.4 LTS, but I can't see any reason why this wouldn't work on newer (or even older) versions of Ubuntu.

**Install**

qBittorrent-nox is already included in the official Ubuntuinto repositories.
So you have to install the package by issuing the following command:
`sudo apt-get update`

`sudo apt-get install qbittorrent-nox`

Alternatively you can use the ppa maintained by surfer-nsk:

`sudo add-apt-repository ppa:surfernsk/internet-software`

`sudo apt-get update`

and proceed to install:

`sudo apt-get install qbittorrent-nox`

**Initscript**

Next up you'll want to setup an initscript to maintain the qbittorrent-nox daemon and logging. First we create the file:

`sudo touch /etc/init.d/qbittorrent`

Now open up this file with a text editor to copy the contents in:

`sudo nano /etc/init.d/qbittorrent`

Once there, paste the following initscript into the file:
```
#! /bin/sh
### BEGIN INIT INFO
# Provides:          qbittorrent-nox
# Required-Start:    $remote_fs $syslog
# Required-Stop:     $remote_fs $syslog
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Starts QBittorrent
# Description:       Start qbittorrent-nox on start. Change USER= before running
### END INIT INFO

# Author: Jesper Smith
# Added log support and cleanup by DementedZA

#Edit the user that qbittorrent-nox will run as.
USER="qbtuser"

# No need to edit these. Logging is sent to /var/log/qbittorrent-nox.log by default.
PATH="/sbin:/usr/sbin:/bin:/usr/bin"
SCRIPTNAME="/etc/init.d/qbittorrent"
NAME="qbittorrent-nox"
DESC="QBittorrent"
PIDFILE="/var/run/$NAME.pid"
QBTLOG="/var/log/$NAME.log"

DAEMON="/usr/bin/qbittorrent-nox"
DAEMON_ARGS=""
DAEMONSTRING="$DAEMON >> $QBTLOG 2>&1"

export DBUS_SESSION_BUS_ADDRESS=""

# Exit if the package is not installed
[ -x "$DAEMONPATH$DAEMON" ] || exit 0

# Read configuration variable file if it is present
#[ -r /etc/default/$NAME ] && . /etc/default/$NAME

# Load the VERBOSE setting and other rcS variables
. /lib/init/vars.sh

# Define LSB log_* functions.
# Depend on lsb-base (>= 3.0-6) to ensure that this file is present.
. /lib/lsb/init-functions

#
# Function that starts the daemon/service
#
do_start()
{

	#Check for log file. If it doesn't exist, create it.
	if [ -f $QBTLOG ];
	then
        	echo "Logging to $QBTLOG."
	else
        	echo "Log file $QBTLOG doesn't exist. Creating it..."
	        touch $QBTLOG
	        chown $USER:$USER $QBTLOG
        	if [ -f $QBTLOG ];
                	then
                        	echo "Logfile created. Logging to $QBTLOG"
	                else
        	                echo "Couldn't create logfile $QBTLOG. Please investigate."
	        fi
	fi

        # Return
        #   0 if daemon has been started
        #   1 if daemon was already running
        #   2 if daemon could not be started

	start-stop-daemon --start --chuid $USER --test --quiet --make-pidfile --pidfile $PIDFILE --background --exec /bin/bash -- -c "$DAEMONSTRING" || return 1

	start-stop-daemon --start --chuid $USER --make-pidfile --pidfile $PIDFILE --background --exec /bin/bash -- -c "$DAEMONSTRING" || return 2
	sleep 1
}

#
# Function that stops the daemon/service
#
do_stop()
{
	start-stop-daemon --stop --exec "$DAEMONPATH$DAEMON"
	sleep 2
	return "$?"
}


case "$1" in
  start)
	[ "$VERBOSE" != no ] && log_daemon_msg "Starting $DESC" "$NAME"
	do_start
	case "$?" in
		0|1) [ "$VERBOSE" != no ] && log_end_msg 0 ;;
		2) [ "$VERBOSE" != no ] && log_end_msg 1 ;;
	esac
	;;
  stop)
	[ "$VERBOSE" != no ] && log_daemon_msg "Stopping $DESC" "$NAME"
	do_stop
	case "$?" in
		0|1) [ "$VERBOSE" != no ] && log_end_msg 0 ;;
		2) [ "$VERBOSE" != no ] && log_end_msg 1 ;;
	esac
	;;
  status)
       status_of_proc "$DAEMON" "$NAME" && exit 0 || exit $?
       ;;
  restart|force-reload)
	log_daemon_msg "Restarting $DESC" "$NAME"
	do_stop
	case "$?" in
	  0|1)
		do_start
		case "$?" in
			0) log_end_msg 0 ;;
			1) log_end_msg 1 ;; # Old process is still running
			*) log_end_msg 1 ;; # Failed to start
		esac
		;;
	  *)
	  	# Failed to stop
		log_end_msg 1
		;;
	esac
	;;
  *)
	#echo "Usage: $SCRIPTNAME {start|stop|restart|reload|force-reload}" >&2
	echo "Usage: $SCRIPTNAME {start|stop|status|restart|force-reload}" >&2
	exit 3
	;;
esac
```
Save the file. Once you've created and saved the script, make it executable and make sure it's owned by root.

```
sudo chmod 755 /etc/init.d/qbittorrent
sudo chown root:root /etc/init.d/qbittorrent
```

**User Configuration**

Go back to the top of the file and edit the user that qbittorrent will run as. Replace qbtuser with the user you would like qbittorrent to run as. Alternatively you could create the user qbtuser by issuing this command: 

`sudo useradd qbtuser` 

now save and exit by pressing CTRL + X, then press 'y' and ENTER.

**Initialization with first start**

Now you'll want to run qbittorrent to get it to generate the config file and show its disclaimer, which you'll need to read and accept in order to continue. This command should be run as the user that will run the init script (qbtuser in our example):

```
sudo su qbtuser
sudo qbittorrent-nox
```

You'll see the following:

```
*** Legal Notice ***
qBittorrent is a file sharing program. When you run a torrent, its data will be made available to others by means of upload. Any content you share is your sole responsibility.

No further notices will be issued.

Press 'y' key to accept and continue...
```
Press 'y' key and press enter. You should see the following:

```
******** Information ********
To control qBittorrent, access the Web UI at http://localhost:8080
The Web UI administrator user name is: admin
The Web UI administrator password is still the default one: adminadmin
This is a security risk, please consider changing your password from program preferences.
17/04/2014 16:52:29 - The Web UI is listening on port 8080
17/04/2014 16:52:30 - qBittorrent is successfully listening on interface 0.0.0.0 port: TCP/6881
17/04/2014 16:52:30 - qBittorrent is successfully listening on interface :: port: TCP/6881
17/04/2014 16:52:35 - External IP: XXX.XXX.XXX.XXX
```
Now that first run is accomplished, you can log in to your server's IP address on port 8080. Say my server's IP is 192.168.0.1, even though it says 0.0.0.0 its bound to your server ip, you can log in by going to http://192.168.0.1:8080
Username is 'admin'
Password is 'adminadmin'

Once you have logged in and see that its working, you can proceed. Go back to the terminal and terminate the process by pressing 'CTRL+C'. Return to your normal user by exiting

`exit`

You can now start qbittorrent with:

**Starting the Daemon**

`sudo service qbittorrent start`

**Configuration**

Now you can log in to the web-server again and proceed to edit your config. If you need to get access to the config file:

`sudo nano /home/qbtuser/.config/qBittorrent/qBittorrent.conf`

You can also monitor the running log with:

**Logging**

`sudo tail -f /var/log/qbittorrent-nox.log`

You can change logging location in the init script by editing the QBTLOG variable.

**Running at Startup**

If you would like qbittorrent to run at startup, update the initscripts with:

`sudo update-rc.d qbittorrent defaults`

**Uninstall**

To uninstall qbittorrent-nox:
```
sudo update-rc.d -f qbittorrent remove #Disables the startup scrupt from startup
sudo apt-get remove --purge qbittorrent* #Removes the filles installed with qbittorrent
sudo rm /etc/init.d/qbittorrent #Removes the init script
sudo rm /var/log/qbittorrent-nox.log #Removes the log file
sudo rm -R /home/qbtuser/.config/qBittorrent #Removes config. You may have reason to keep this
sudo add-apt-repository -r ppa:surfernsk/internet-software # Removes the PPA if you used it.
```
