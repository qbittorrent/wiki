You locked the qBittorrent user interface and forgot the password you used?
Don't worry, you can recover from this.

1. Exit or kill qBittorrent process.

2. Open the preference file with a text editor.<br/>
Windows<br/>
`%APPDATA%\qBittorrent\qbittorrent.ini` = `C:\Users\<username>\AppData\Roaming\qBittorrent\qbittorrent.ini`<br/>
Linux<br/>
`~/.config/qBittorrent/qBittorrent.conf`<br/>
macOS<br/>
`~/.config/qBittorrent/qbittorrent.ini`<br/>

3. Remove the following lines:
    ```ini
    [Locking]
    password=<encrypted_password>         # this only appear on qBittorrent version < 4.2.0
    password_PBKDF2=<encrypted_password>  # this only appear on qBittorrent version ≥ 4.2.0
    locked=true
    ```

4. Save the file

5. Start qBittorrent again


## Before qBittorrent v3.3.6 on macOS
1. Exit or kill qBittorrent process

2. Open a terminal and run the following command:
    ```shell
    defaults delete com.qbittorrent.qBittorrent Locking.locked
    defaults delete com.qbittorrent.qBittorrent Locking.password
    ```

3. Start qBittorrent again
