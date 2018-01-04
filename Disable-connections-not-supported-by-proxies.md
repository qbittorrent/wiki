**Note**: a proxy must be configured in order to make use of this feature.

This feature attempts to enhance client privacy by disallowing any traffic that doesn't flow through a configured proxy. When enabled, the following configuration is used:

* Disable Local Peer Discovery
* Disable DHT
* Disable UPnP & NAT-PMP
* Only talk to http(s) trackers via (any) proxy
* Only talk to udp trackers via SOCKS5/I2P proxy