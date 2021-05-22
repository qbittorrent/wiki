
## Always enabled in qBittorrent 4.2 and above

[Changes in the underlying libtorrent library](https://github.com/qbittorrent/qBittorrent/issues/11698#issuecomment-567733276) mean that the below behavior is always enabled when using a proxy.

The option is removed.

## Before qBittorrent 4.2

**Note**: a proxy must be configured in order to make use of this feature.

This feature attempts to enhance client privacy by disallowing any traffic that doesn't flow through a configured proxy. If enabling this option, you may also be interested in [Anonymous Mode](Anonymous-Mode).

## Features
* Disables Local Peer Discovery
* Disables UPnP & NAT-PMP
* Only talks to http(s) trackers via (any) proxy
* Only talks to udp trackers via SOCKS5/I2P proxy
