Folks running qBittorrent on a linux desktop can easily setup pop-up alerts (among other things) triggered when a torrent download finishes. For the purpose of this example, let's assume a pop-up notice is the goal.

The libnotify package should have both "notify-send" and "libnotify," the former being a CLI to the later.  To incorporate it into your qBittorrent configuration, open the "Options" window and select the "Downloads" page.  Enable the "Run an external program on torrent completion" check-box and enter the following into the command text-box:

/usr/bin/notify-send -a /usr/bin/qbittorrent -i /usr/share/icons/hicolor/96x96/apps/qbittorrent.png -t 15000 "qBT Download Complete" "%n"

This will cause a notification bubble containing the qBT icon, message and torrent name to display for 15 seconds.  Of course you may want/need to change the paths or other settings to suit your needs.