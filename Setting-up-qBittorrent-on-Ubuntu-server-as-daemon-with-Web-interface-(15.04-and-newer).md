### Introduction: ###
qBittorrent has a feature-rich Web UI allowing users to control qBittorrent remotely. This is ideal for headless servers without the X window system such as Ubuntu Server.


### Note: ###
Since Ubuntu (Server) 15.04, upstart was replaced with systemd. Thus, this guide only applies to Ubuntu 15.04 and newer. This change only affects how qbittorrent is started (init scripts) and how logging happens.


### A little about qBittorent-nox vs qBittorrent: ###
The main binary file (executable file) for qBittorrent has two variants and is usually located at /usr/bin/:
* qbittorrent which is qBittorrent *with* Graphical user interface and 
* qbittorrent-nox which is same as the above but compiled *without* the desktop graphical user interface, and only a web-interface.   
  
Since this guide focuses on servers that normally don't have a desktop user interface, we will use the qbittorrent-nox binary, but we still call it by it's regular name - qbittorrent.


### Install ###
qBittorrent is now available in official Ubuntu repositories.
More up-to-date packages are published on our [stable](https://launchpad.net/~qbittorrent-team/+archive/ubuntu/qbittorrent-stable) and [unstable](https://launchpad.net/~qbittorrent-team/+archive/ubuntu/qbittorrent-unstable) PPAs. 
The stable PPA supports Ubuntu 14.04 LTS (only the libtorrent-rasterbar package), 16.04 LTS, 17.04, 17.10, 18.04 LTS, and 19.10. 

Quick instructions
To use these PPAs please use the following command and make sure your version is supported:

qBittorrent Stable
```
$ sudo add-apt-repository ppa:qbittorrent-team/qbittorrent-stable
```

Then install qBittorrent by doing this:
```
$ sudo apt-get update && sudo apt-get install qbittorrent-nox
```


### User ###
Assuming you want qbittorent to run as its own user (better for security purposes), let's create the user that qbittorrent will run under and give it a password when prompted. Just press `Return` for values you want to leave blank:

```
sudo adduser qbtuser
Adding user `qbtuser' ...
Adding new group `qbtuser' (1003) ...
Adding new user `qbtuser' (1003) with group `qbtuser' ...
Creating home directory `/home/qbtuser' ...
Copying files from `/etc/skel' ...
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
Changing the user information for qbtuser
Enter the new value, or press ENTER for the default
        Full Name []:
        Room Number []:
        Work Phone []:
        Home Phone []:
        Other []:
Is the information correct? [Y/n] y
```


### Create Service ###
First, create a file:
`/etc/systemd/system/qbittorrent.service`
```
[Unit]
Description=qBittorrent Daemon Service
After=network.target

[Service]
User=qbtuser
Group=qbtuser
ExecStart=/usr/bin/qbittorrent-nox
ExecStop=/usr/bin/killall -w qbittorrent-nox

[Install]
WantedBy=multi-user.target
```

Normally after editing services we'd issue a reload command but since it will also invoke qbittorrent before we initialized the configuration, we'll give it a skip for now. If you ever make changes to the services file, update systemctl with:
```
$ sudo systemctl daemon-reload
```


### Initializing Configuration ###

Before we continue, let's run qbittorrent so that it can ask us to accept the disclaimer, and save and create the config file to remember this setting. Doing this will create and save the configuration files under:
```
/home/qbtuser/.config/qBittorrent/
```

First we impersonate the qbtuser:
```
$ sudo su qbtuser
```

And run qbittorrent:  
```
$ qbittorrent-nox
```

You should see the following:  

    *** Legal Notice ***
    qBittorrent is a file sharing program. When you run a torrent, its data will be made available to others by means of upload. Any content you share is your sole responsibility.
    
    No further notices will be issued.
    
    Press 'y' key to accept and continue...


Press y key and you should see:

    ******** Information ********
    To control qBittorrent, access the Web UI at http://localhost:8080
    The Web UI administrator user name is: admin
    The Web UI administrator password is still the default one: adminadmin
    This is a security risk, please consider changing your password from program preferences.
    03/02/2016 15:51:03 - The Web UI is listening on port 8080
    03/02/2016 15:51:04 - qBittorrent is successfully listening on ...

Without doing anything further, you should be able to point a web browser (on the same network) to the IP of your server: http://ip-of-server:8080 and log in with:  

    User: admin
    Password: adminadmin

You should now see the web interface.

Back at the server's command line, exit out of qbittorrent-nox instance with `Ctrl-c`

Now, stop impersonating the qbittorrent user to return to our account with sudo access:
```
$ exit
```

### Enable service: ###
Now start the service,
```
$ sudo systemctl start qbittorrent
```

Verify it is running:
```
$ sudo systemctl status qbittorrent
```

Quit out of the above screen by pressing `q`

Enable it to start up on boot:
```
$ sudo systemctl enable qbittorrent
```

and you should see:
```
Created symlink from /etc/systemd/system/multi-user.target.wants/qbittorrent.service to /etc/systemd/system/qbittorrent.service.
```


### That's it, we are done! ###
After the above, systemd should have indexed and invoked our init script so qbittorrent should be running. qBittorrent should now start automatically with reboots.


### Disable account login ###
You may also want to issue the following command to disable login for the account (from SSH) for security reasons. The account will still be usable locally:
```
$ sudo usermod -s /usr/sbin/nologin qbtuser
```

This can be reversed if necessary with the command:
```
$ sudo usermod -s /bin/bash qbtuser
```


### Starting qBittorrent ###
```
$ sudo systemctl start qbittorrent
```


### Stopping qBittorrent: ###
```
$ sudo systemctl stop qbittorrent
```


### Check status: ###
```
$ sudo systemctl status qbittorrent
```

This should yield something like this:

    ● qbittorrent.service - qBittorrent Daemon Service
       Loaded: loaded (/etc/systemd/system/qbittorrent.service; disabled; vendor preset: enabled)
       Active: active (running) since Wed 2016-02-03 16:02:06 SAST; 6s ago
      Process: 15469 ExecStart=/usr/bin/qbittorrent-nox -d (code=exited, status=0/SUCCESS)
     Main PID: 15470 (qbittorrent-nox)
       CGroup: /system.slice/qbittorrent.service
               └─15470 /usr/bin/qbittorrent-nox -d

To determine the status of qBittorrent, pay special attention to:

       Active: active (running) since Wed 2016-02-03 16:02:06 SAST; 6s ago'


### Logging and debugging: ###
With the advent of systemd replacing upstart, logging also changed. qbittorrent doesn't have a straightforward logging facility. When it runs it outputs to syslog, but when it doesn't run, like if the disclaimer hasn't been answered yet, you won't see anything in the log files. The best way to see this is to impersonate the qbtuser and run qbittorent-nox to see if the disclaimer comes up again.

Another way of working around how qbittorrent logs/doesn't log is to modify the init script as a 'oneshot' command execution and pipe the output to a file such as `/var/log/qbittorrent.log`, but I haven't tried this yet. Possibly the process identifier will need to be specified as well as the stop command so that status and stop commands also work. This is still a work in progress and I'll update this guide when I have it working.

You can also view output of qbittorrent with journalctl:  
This will show the entire log in a pager that can be scrolled through:
```
$ sudo journalctl -u qbittorrent.service
```

This will show the live version of the log file as things are happening:
```
$ sudo journalctl -f -u qbittorrent.service
```


### Uninstalling qBittorrent: ###
Remove the startup script:
```
$ sudo rm /etc/systemd/system/qbittorrent.service
```

Remove the qBittorrent package:
```
$ sudo apt-get remove --purge "qbittorrent*"
```

Remove log files:  
```
$ sudo rm /var/log/qbittorrent.log
```

Remove qbtuser's home folder and config files. If you want to re-install qbittorrent or something you might want to keep this or back it up:
```
$ sudo rm -R /home/qbtuser
```

Remove qbtuser:  
```
$ sudo userdel qbtuser
```

### (Optional) Service Dependencies ###

Let's say that you configured qbittorrent-nox to download files to a directory that is in another drive, for example, mounted on '/media/volume'. It's important that you edit the service created above and add some systemd dependencies to prevent qbittorrent from writing files on a directory that it should not, in case your drive fails to mount or is accidentally unmounted.

After you added the mount point to the `/etc/fstab`, it should have a line like this:
```
UUID=c987355d-0ddf-4dc7-bbbc-bab8989d0690 /media/volume  ext4     defaults,nofail 0       0
```
The 'nofail' option prevents the system from stopping the boot process in case the drive can't mount on startup.

You should edit `/etc/systemd/system/qbittorrent.service` to add 'local-fs.target' to the line 'After=network.target' and add the line 'BindsTo=media-volume.mount' to bind the qbittorrent service to the mount point that you want it to write the files. Your service file should look like this: 
```
[Unit]
Description=qBittorrent Daemon Service
After=network.target local-fs.target
BindsTo=media-volume.mount

[Service]
User=qbtuser
Group=qbtuser
ExecStart=/usr/bin/qbittorrent-nox
ExecStop=/usr/bin/killall -w qbittorrent-nox

[Install]
WantedBy=multi-user.target
```

The 'media-volume.mount' is a systemd unit created dynamically at boot, based on the entries found on `/etc/fstab`
and is used by systemd to mount and manage the status of the respective drive. [Systemd.mount reference](http://man7.org/linux/man-pages/man5/systemd.mount.5.html)
It follows a simple logic: if your drive is mounted on '/media/volume', the unit name will be 'media-volume.mount', if it's on '/mnt/disk', the unit will be 'mnt-disk.mount'.

This way, if the drive can't mount on boot or if the drive is unmounted after qbittorrent started, it will not allow it to start or force it to stop, preventing from writing when the drive is not ready or present.