You locked the qBittorrent user interface and forgot the password you used?
Don't worry, you can recover from this.

## Windows
1. Exit or kill qBittorrent process
2. Open `qbittorrent.ini` with a text editor, refer to [here](https://github.com/qbittorrent/qBittorrent/wiki/Frequently-Asked-Questions#Where_does_qBittorrent_save_its_settings) to see where it is located.
3. Remove the following lines:

 ```ini
[Locking]
password=<encrypted_password>
locked=true
```
4. Save the file
5. Start qBittorrent again

## Linux
1. Exit or kill qBittorrent process
2. Open `qBittorrent.conf` with a text editor, refer to [here](https://github.com/qbittorrent/qBittorrent/wiki/Frequently-Asked-Questions#Where_does_qBittorrent_save_its_settings) to see where it is located.
3. Remove the following lines:

 ```ini
[Locking]
password=<encrypted_password>
locked=true
```
4. Save the file
5. Start qBittorrent again

## Mac OS X
1. Exit or kill qBittorrent process
2. Open a terminal and run the following command:

 ```shell
defaults delete com.qbittorrent.qBittorrent Locking.locked
defaults delete com.qbittorrent.qBittorrent Locking.password
```
3. Start qBittorrent again
