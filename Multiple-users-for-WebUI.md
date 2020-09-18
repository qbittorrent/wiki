This is easy to accomplish, follow [this](https://github.com/qbittorrent/qBittorrent/wiki/Running-qBittorrent-without-X-server-(WebUI-only---systemd-service-setup,-Ubuntu-15.04-or-newer)#setup-the-systemd-service) step to create a service.

To have multiple users repeat this step multiple times, though you must give each file a different name:

```sh
sudoedit /etc/systemd/system/qbittorrent.service
```

`qbittorrentUSER1.service` `qbittorrentUSER2.service` `qbittorrentUSER3.service`

You must also change the user the service runs under so that you can have multiple instances of qbittorrent running at once.

```
# change user as needed
User=qbtuser
```