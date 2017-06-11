If your user name and password is locked out of the Web UI and the default user name `admin` and password `adminadmin` is also failing, the steps to recover access are:

1. Log in to the machine the client is on as the user that qbittorrent runs as, either at the console or over a SSH connection.
2. Navigate to `~/.config/qBittorrent/` and edit the qBittorrent configuration file `qBittorrent.conf`.
3. Locate the entry of `WebUI\Password_ha1` and change it to this:

 ```ini
WebUI\Password_ha1=@ByteArray(5ebe2294ecd0e0f08eab7690d2a6ee69)
```
the password will be set to `secret`.
4. You should then be able to login the WebUI using your usual username and the password `secret`. Remember to change it back to your preferred password.


You could also try copy/pasting the existing value in WebUI\Password_ha1=@ByteArray(value) into https://isc.sans.edu/tools/reversehash.html and that could possibly reveal what the original password was.