qBittorrent supports a feature called "Disable connections not supported by proxies"

This feature attempts to enhance privacy by only allowing network traffic to flow via a configured proxy. When enabled, the following configuration is used:

* Disable Local Peer Discovery
* Disable DHT
* Disable UPnP & NAT-PMP
* Only talk to http(s) trackers via any configured proxy
* Only talk to udp trackers via SOCKS5/I2P proxy
