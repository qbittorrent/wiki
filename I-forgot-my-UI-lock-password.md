You locked qBittorrent user interface and forgot the password you used?
Don't worry, you can recover from this.

## Windows / Linux
* Kill qBittorrent
* Look for qBittorrent.conf on your hard disk (_~/.config/qBittorrent/qBittorrent.conf_ on Linux)
* Remove the following lines:
```c++
[Locking]
password=<encrypted_password>
locked=true
```
* Save the file
* Start qBittorrent again

## Mac OS X
* Kill qBittorrent
* Open a terminal and type the following:
``rm $HOME/Library/Preferences/org.qbittorrent.plist*``
* Start qBittorrent again

The user interface should not be locked anymore. Next time you lock it, it will ask you to choose a password again.