You locked qBittorrent user interface and forgot the password you used?
Don't worry, you can recover from this.

## Windows
* Kill qBittorrent
* Look for **qbittorrent.ini** file on your hard disk. It is usually located under _C:\Documents and Settings\USERNAME\Application Data\qBittorrent_ or _C:\Users\USERNAME\AppData\qBittorrent_
* Remove the following lines:
```c++
[Locking]
password=<encrypted_password>
locked=true
```
* Save the file
* Start qBittorrent again

## Linux
* Kill qBittorrent
* Edit _~/.config/qBittorrent/qBittorrent.conf_
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
``rm $HOME/Library/Preferences/com.qbittorrent.plist*``
* Start qBittorrent again

The user interface should not be locked anymore. Next time you lock it, it will ask you to choose a password again.