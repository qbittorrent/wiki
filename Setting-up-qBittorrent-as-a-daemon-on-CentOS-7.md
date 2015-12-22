# Platform
* CentOS 7_x64
* systemctl

# Install
You have two options for installation

* One, follow this guide [Compiling qbittorrent-nox for CentOS from source](https://github.com/qbittorrent/qBittorrent/wiki/Compiling-qbittorrent-nox-for-CentOS-from-source)
* Two, install from [EPEL](https://fedoraproject.org/wiki/EPEL)

If you choose to install from EPEL you can simply `sudo yum install qbittorrent-nox.x86_64`

# Creating a systemctl script
1. Create a script so that we can control qbittorrent with systemctl  
`sudo vi /usr/lib/systemd/system/qbittorrent.service`  
2. Enter`i`to enter insert mode then paste the code below, change **\<username\>** to the user you want to run qbittorrent with. Note that you can't run as user `nobody`.    
```
    [Unit]
    Description=qbittorrent torrent server
    
    [Service]
    User=<username>
    ExecStart=/usr/bin/qbittorrent-nox
    Restart=on-abort
    
    [Install]
    WantedBy=multi-user.target
```
3. Save the script  
Enter`:wq` then press enter to save the script  
4. reload systemctl, this will make systemd take notice of the new script  
`sudo systemctl daemon-reload`  
5. Start qbittorrent and accept the user agreement
`sudo qbittorrent-nox`, press `y` then `Enter`. Now close qbittorrent `Ctrl-C`
5. Start the qbittorrent service
`sudo systemctl start qbittorrent`  

# Start at boot
`sudo systemctl enable qbittorrent`

# Source
https://stackoverflow.com/a/26565328