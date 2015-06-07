# Platform
* CentOS 7_x64
* systemctl

# Install
Follow this guide [Compiling qbittorrent-nox for CentOS from source](wiki/Compiling-qbittorrent-nox-for-CentOS-from-source)

# Creating a systemctl script
1. Create a script so that we can control qbittorrent with systemctl  
`sudo vi /usr/lib/systemd/system/qbitorrent.service`  
2. Enter`i`to enter insert mode then paste the code below, change **\<username\>** to the user you want to run qbittorrent with    
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
5. Start qbittorrent  
`sudo systemctl start qbittorrent`  

# Start at boot
`sudo systemctl enable qbittorrent`

# Source
http://stackoverflow.com/a/26565328