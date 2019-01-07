If your user name and password is locked out of the Web UI and the default user name `admin` and password `adminadmin` is also failing, the steps to recover access are:

1. Log in to the machine the client is on as the user that qbittorrent runs as, either at the console or over an SSH connection.

2. Navigate to `~/.config/qBittorrent/` and edit the qBittorrent configuration file `qBittorrent.conf`.

    * If you're using qbittorrent version < 4.2.0, locate the entry of `WebUI\Password_ha1` and remove the entire line.
    * If you're using qbittorrent version ≥ 4.2.0, locate the entry of `WebUI\Password_PBKDF2` and remove the entire line.

3. Save and close the file and restart qBittorrent.

4. You should then be able to login the WebUI using your usual username and the default password `adminadmin`.

5. If you are still unable to login check if your antivirus is not blocking sending password via unencrypted connection (http). Bitdefender may do it by default

**Remember to change it back to your preferred password.**
