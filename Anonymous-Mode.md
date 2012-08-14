Since v2.9.0, qBittorrent supports a feature called "Anonymous Mode".

When enabled, qBittorrent tries to hide its identity to a certain degree. The peer-ID will no longer include the client's fingerprint. The user-agent will be reset to an empty string. Trackers will only be used if they are using a proxy server. The listen sockets are closed, and incoming connections will only be accepted through a SOCKS5 or I2P proxy (if a peer proxy is set up and is run on the same machine as the tracker proxy). Since no incoming connections are accepted, NAT-PMP, UPnP, DHT and local peer discovery are all turned off when this setting is enabled.

If you're using I2P, it might make sense to enable anonymous mode as well.