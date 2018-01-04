Since v2.9.0, qBittorrent supports a feature called "Anonymous Mode".

When enabled, qBittorrent tries to hide its identity to a certain degree:
* The peer-ID will no longer include the client's fingerprint.
* The user-agent will be reset to an empty string.
* Other identifying information will not be leaked, such as your local listen port, your IP etc.

If you're using I2P, a VPN or a proxy, it might make sense to enable anonymous mode.
