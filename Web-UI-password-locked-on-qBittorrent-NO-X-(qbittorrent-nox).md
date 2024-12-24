If your user name and password is locked out of the WebUI and the default user name `admin` and password `adminadmin` is also failing, the steps to recover access are:

1. Log in to the machine the client is on as the user that qbittorrent runs as, either at the console or over an SSH connection.

2. Exit (or kill) qBittorrent process.

3. Navigate to `~/.config/qBittorrent/` (or `/<qbittorrent user name>/.config/qBittorrent/`) and edit the configuration file `qBittorrent.conf`.
    * If you're using qbittorrent version ≥ 4.2.0, locate the line starting with `WebUI\Password_PBKDF2` and remove the entire line.
    * If you're using qbittorrent version < 4.2.0, locate the line starting with `WebUI\Password_ha1` and remove the entire line.

4. Save and close the file and start qBittorrent.

5.
   * If you're using qbittorrent version ≥ 4.6.1, run qbittorrent in the console and it will present you with a temporary password. Use it to log in WebUI.
   * If you're using qbittorrent version < 4.6.1, you are able to login WebUI with your usual username and the default password `adminadmin`.

6. ***Remember to update it to your preferred password, otherwise your machine is prone to be hacked.***

ps. If you are still unable to login check if your antivirus is not blocking sending password via unencrypted connection (http). Bitdefender seems to do it by default, you need to whitelist qbittorrent server URL.
