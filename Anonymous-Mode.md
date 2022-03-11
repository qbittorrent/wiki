**WARNING: anonymous mode doesn't provide strong privacy guarantees on its own. If you are concerned about legal authorities and copyright trouble, for example, consider using a VPN instead (or in addition to it). Anonymous mode is only meant to further prevent your BitTorrent traffic from being associated to you, even when using other privacy enhancing mechanisms (such as a VPN service), by limiting the scope of the information broadcasted by the client (such as the fingerprint in the user-agent).**

When enabled, qBittorrent will take certain measures to try to mask its identity. The exact functionality of Anonymous Mode depends on the version of libtorrent-rasterbar being used and whether the torrent is private or not.

If you're using VPN, proxy or I2P, you probably want to enable Anonymous Mode.

## qBittorrent 4.4.x with libtorrent 2.0.6+

* A generic user-agent will be used for trackers (except for private torrents).
* Your local IPv4 and IPv6 addresses won't be sent as query string parameters to private trackers(requires private torrent). So a tracker will not get your IPv6 address from IPv4 announce and vice versa. The tracker will only see the IP address that you are announcing from.
* If `IP address reported to tracker` is configured, it will not be sent to trackers.
* The client version will not be sent to peers in the extension handshake.

## qBittorrent v3.3.0+

* The peer-ID will no longer include the client's fingerprint
* The user-agent will be reset to an empty string
* Other identifying information will not be exposed to the public directly, such as IP, listening port, etc.
* The announced port used to be `0` but changed to `1` for versions using libtorrent 1.2.5 or later, since some trackers would fail the announce for `0`.

You may enable additional identity-masking features with the ["Disable connections not supported by proxies"](Disable-connections-not-supported-by-proxies) option.

## qBittorrent v2.9.0 - v3.2.5

These versions of qBittorrent may make use of a version of libtorrent-rasterbar below 1.0.0. If so, enabling Anonymous Mode will enable all of the aforementioned features, plus some additional ones:

* Disables Local Peer Discovery
* Disables DHT
* Disables UPnP & NAT-PMP
* Only talks to http(s) trackers via (any) proxy
* Only talks to udp trackers via SOCKS5/I2P proxy

These additional features have since been migrated to the "Disable connections not supported by proxies" option in later versions of qBittorrent.
