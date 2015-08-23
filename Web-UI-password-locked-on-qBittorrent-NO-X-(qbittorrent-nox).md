Relevant to all client versions running on a Linux OS without a GUI (No X-Windows)

If your user name and password is locked out of the Web UI and the default user name (admin) and password (adminadmin) is also failing the steps to recover access are:

Log in to the machine the client is on as the user that qbittorrent runs as, either at the console or over a SSH connection.

Navigate to ~/.config/qBittorrent/ and edit the qBittorrent configuration file, (qBittorrent.conf);

Locate the entry of WebUI\Password_ha1 and change the ByteArray value to,
 "5ebe2294ecd0e0f08eab7690d2a6ee69" 
(without the quotes) and the password will be set to "secret" (without the quotes).

You should then be able to login the WebUI using your usual username and the password above and change it back to your preferred password.

