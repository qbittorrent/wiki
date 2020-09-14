If your user name and password is locked out of the Web UI and the default user name `admin` and password `adminadmin` is also failing, the steps to recover access are:

1. Log in to the machine the client is on as the user that qbittorrent runs as, either at the console or over an SSH connection.

2. Exit or kill qBittorrent process.

3. Navigate to `~/.config/qBittorrent/` or `/home/qbittorrent-nox/.config/qBittorrent/`and edit the qBittorrent configuration file `qBittorrent.conf`.
    * If you're using qbittorrent version < 4.2.0, locate the entry of `WebUI\Password_ha1` and remove the entire line.
    * If you're using qbittorrent version ≥ 4.2.0, locate the entry of `WebUI\Password_PBKDF2` and remove the entire line.

4. Save and close the file and restart qBittorrent.

5. You should then be able to login the WebUI using your usual username and the default password `adminadmin`.

ps. If you are still unable to login check if your antivirus is not blocking sending password via unencrypted connection (http). Bitdefender seems to do it by default, you need to whitelist qbittorrent server URL.

**Remember to change it back to your preferred password.**
