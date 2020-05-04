# Introduction

Since version 2.5.0, qBittorrent ships a fully-functional built-in tracker that allows you to share files with your friends without having to use public trackers.

# How to enable the embedded tracker

qBittorrent's embedded tracker is disabled by default.
You can enable it in the advanced settings (`Tools -> Options -> Advanced -> Enable embedded tracker`).

By default, the tracker URL is `http://your_public_ip:9000/announce`.
You can check [this website](http://www.whatsmyip.org/) to know your public IP address.

# Forward the tracker port on your router

Your home connection is most likely behind some kind of NAT, so if you want your tracker to be accessible from Internet (i.e. outside your LAN), you need to configure your router to forward the tracker port to your machine.
Otherwise, your friends will not be able to connect to it.
This process of port forwarding is different for every router, so it is not documented here.
There are plenty of guides explaining how to do it on the Internet.

Once you have configured your router, you can test that your configuration is working:

1. Start qBittorrent and enable the embedded tracker
2. Go to a website that can check if your ports are open such as [this one](https://www.yougetsignal.com/tools/open-ports/)
3. Type in the public address and port of your tracker
4. Press the "Check" button
5. If your router is properly configured, you should get a result like `Port <port> is open on address.`

# Sharing files with friends

1. Start qBittorrent and enable the embedded tracker
2. Create a torrent with the files you wish to share; you can use qBittorrent's built-in torrent creator (`Tools -> Torrent creator`)
    1. In "Tracker URLs" field, use the URL of the embedded tracker `http://your_public_ip:9000/announce`
    2. Check the "Start seeding immediately" box
    3. Customize the remaining options to your liking
    4. Press the "Create Torrent" button
3. Send this newly created `.torrent` file to your friends and they will be able to download the files you are sharing using any BitTorrent client.

If you are using `qbittorrent-nox` (qBittorrent without the GUI), you can use a tool such as `mktorrent` to create the torrent.
