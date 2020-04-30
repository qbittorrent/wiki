# Introduction

qBittorrent has a feature-rich Web UI allowing users to control qBittorrent remotely. `qbittorrent-nox` is a version of qBittorrent without any GUI components. This is ideal for headless servers without the X window system such as Ubuntu Server.

This guide will show you how to setup `qbittorrent-nox` to run as a managed background service (daemon) by setting it up as a `systemd` service. It can then be customized like any other `systemd` service, to automatically start on boot, for instance.

Refer to https://github.com/qbittorrent/qBittorrent/wiki/Installing-qBittorrent for more information on how to install the latest version of `qbittorrent-nox` on your system.

For example, on Ubuntu, after adding the PPA simply do `sudo apt install qbittorrent-nox`.

Note: these instructions are written with Ubuntu in mind but should be much the same if not exactly the same for any modern distro that uses `systemd`.

# Create a separate user account (optional)

Assuming you want qbittorent to run as its own user (better for security purposes), let's create the user that qbittorrent will run under and give it a password when prompted.
You may leave every other value blank.

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

### Disable qbtuser account SSH login (optional)

You may also want to disable login for the account (from SSH) for security reasons. The account will still be usable locally:

```
$ sudo usermod -s /usr/sbin/nologin qbtuser
```

This can be reversed if necessary with the command:

```
$ sudo usermod -s /bin/bash qbtuser
```

# Create Service

qBittorrent includes a bundled `systemd` service file since version 3.2.0.

Copy the contents to `/etc/systemd/system/qbittorrent.service` (or the whatever the standard directory for service units is in your distribution) and edit it according to your needs.
The following example is based on the bundled file, and includes some possible modifications you might want to make.

```
[Unit]
Description=qBittorrenti-nox service
Documentation=man:qbittorrent-nox(1)
Wants=network-online.target
After=network-online.target nss-lookup.target

[Service]
# if you have systemd >= 240, you probably want to use Type=exec instead
Type=simple
# change user as needed
User=qbtuser
# notice that no -d flag needed
ExecStart=/usr/bin/qbittorrent-nox
# qBittorrent >= 4.2.x sets open file limits to unlimited by itself, so this is not needed:
LimitNOFILE=infinity

[Install]
WantedBy=multi-user.target
```

Normally after editing services we'd issue a reload command but since it will also invoke qbittorrent before we initialized the configuration, we'll give it a skip for now.
If you ever make changes to the services file, update systemctl with:

```
$ sudo systemctl daemon-reload
```

# Initializing Configuration

Before we continue, let's run qbittorrent so that it can ask us to accept the disclaimer, and save and create the config file to remember this setting. Doing this will create and save the configuration files under:

```
/home/qbtuser/.config/qBittorrent/
```

First we impersonate the qbtuser and `run qbittorrent-nox`:

```
$ sudo su qbtuser
$ qbittorrent-nox
```

You should see the legal disclaimer. Accept it, and qBittorrent should print the URL where the WebUI can be accessed at, as well as the default credentials.

You should now be able to access the WebUI via a Web browser.

Back at the server's command line, stop the `qbittorrent-nox` instance with `Ctrl-c`.

Now, stop impersonating the qbittorrent and return to your account:

```
$ exit
```

# Controlling the service

The qBittorrent service is now ready to be used. It can be controlled like any other `systemd` service:

- start the service: `sudo systemctl start qbittorrent`
- check service status: `sudo systemctl status qbittorrent`
- stop the service: `sudo systemctl stop qbittorrent`
- enable it to start up on boot: `sudo systemctl enable qbittorrent`
    - this should output something like the following:
        ```
        Created symlink from /etc/systemd/system/multi-user.target.wants/qbittorrent.service to /etc/systemd/system/qbittorrent.service.
        ```
- the previous command can be reverted with: `sudo systemctl disable qbittorrent`

# Logging

qBittorrent will still log most interesting stuff to its usual logging directory. In this example, this would be `/home/qbtuser/.local/share/data/qBittorrent/logs/`.

However, some output can probably still be viewed with:

```
$ sudo journalctl -u qbittorrent.service
```

For more information on how to use and customize `systemd` logging, refer to its documentation.

# Service Dependencies (optional)

Let's say that you configured qbittorrent-nox to download files to a directory that is in another drive, for example, mounted on '/media/volume'.
It's important that you edit the service created above and add some `systemd` dependencies to prevent qbittorrent from writing files on a directory that it should not, in case your drive fails to mount or is accidentally unmounted.

After you added the mount point to the `/etc/fstab`, it should have a line like this:
```
UUID=c987355d-0ddf-4dc7-bbbc-bab8989d0690 /media/volume  ext4     defaults,nofail 0       0
```
The 'nofail' option prevents the system from stopping the boot process in case the drive can't mount on startup.

You should edit `/etc/systemd/system/qbittorrent.service` to add 'local-fs.target' to the line 'After=network.target' and add the line 'BindsTo=media-volume.mount' to bind the qbittorrent service to the mount point that you want it to write the files. Your service file should look like this: 
```
# ... other stuff ...

[Unit]
# ... other stuff ...
After=network.target local-fs.target
BindsTo=media-volume.mount

# ... other stuff ...
```

The 'media-volume.mount' is a `systemd` unit created dynamically at boot, based on the entries found on `/etc/fstab`
and is used by `systemd` to mount and manage the status of the respective drive. [Systemd.mount reference](http://man7.org/linux/man-pages/man5/systemd.mount.5.html)
It follows a simple logic: if your drive is mounted on '/media/volume', the unit name will be 'media-volume.mount', if it's on '/mnt/disk', the unit will be 'mnt-disk.mount'.

This way, if the drive can't mount on boot or if the drive is unmounted after qbittorrent started, it will not allow it to start or force it to stop, preventing from writing when the drive is not ready or present.