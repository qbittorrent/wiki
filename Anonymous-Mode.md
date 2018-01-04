When enabled, qBittorrent will take certain measures to try to mask its identity. The exact functionality of Anonymous Mode depends on the version of qbittorrent (specifically, libtorrent) being used.

If you're using I2P, a VPN or a proxy, it is recommended that you enable anonymous mode.

## qBittorrent v3.3.0+

* The peer-ID will no longer include the client's fingerprint.
* The user-agent will be reset to an empty string.
* Other identifying information will not be exposed, such as your local listen port, your IP etc.

You may enable additional identity-masking features with the [Disable connections not supported by proxies](https://github.com/qbittorrent/qBittorrent/wiki/Disable-connections-not-supported-by-proxies) option.

## qBittorrent v2.9.0 - v3.2.5

These versions of qBittorrent may make use of a version of libtorrent below 1.0.0. If so, enabling Anonymous Mode will enable all of the aformentioned features, plus some additional ones:

* Disables Local Peer Discovery
* Disables DHT
* Disables UPnP & NAT-PMP
* Only talks to http(s) trackers via (any) proxy
* Only talks to udp trackers via SOCKS5/I2P proxy

These additional features have since been migrated to the _Disable connections not supported by proxies_ option in later versions of qBittorrent.