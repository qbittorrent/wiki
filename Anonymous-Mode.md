When enabled, qBittorrent will take certain measures to try to mask its identity. The exact functionality of Anonymous Mode depends on the version of qBittorrent (specifically, libtorrent-rasterbar) being used.

If you're using VPN, proxy or I2P, you probably want to enable Anonymous Mode.

## qBittorrent v3.3.0+

* The peer-ID will no longer include the client's fingerprint
* The user-agent will be reset to an empty string
* Other identifying information will not be exposed, such as IP, listening port, etc.

You may enable additional identity-masking features with the ["Disable connections not supported by proxies"](Disable-connections-not-supported-by-proxies) option.

## qBittorrent v2.9.0 - v3.2.5

These versions of qBittorrent may make use of a version of libtorrent-rasterbar below 1.0.0. If so, enabling Anonymous Mode will enable all of the aforementioned features, plus some additional ones:

* Disables Local Peer Discovery
* Disables DHT
* Disables UPnP & NAT-PMP
* Only talks to http(s) trackers via (any) proxy
* Only talks to udp trackers via SOCKS5/I2P proxy

These additional features have since been migrated to the "Disable connections not supported by proxies" option in later versions of qBittorrent.
