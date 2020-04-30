# Introduction

On Linux, the `qbittorrent-nox` package includes a version of qBittorrent that does not require any GUI-related components to run, ideal to run in headless server environments. It can be controlled via a feature rich HTTP WebAPI and WebUI.

You can install it from a binary package repository such as your distro's repositories, the official Ubuntu PPAs, etc.

You can also compile it from source by toggling the flag that disables the GUI (refer to the relevant compilation guides on the wiki).

# Running

Simply execute `qbittorrent-nox`. At first execution, you should get prompted to accept the legal disclaimer. After that, qBittorrent will print out some useful information, such as the URL the WebAPI/WebUI can be accessed at, via a web browser, and the default credentials.

To quit, simply press `Ctrl-c` or send `SIGINT` or `SIGTERM` to the process.

### Running in the background

After first start and having accepted the legal disclaimer, you can execute `qbittorrent-nox -d` to execute qbittorrent as a daemon.

To control the daemonized process, checkout https://github.com/qbittorrent/qBittorrent/wiki/Running-qBittorrent-as-a-daemon-without-X-server-(systemd-distros-like-Ubuntu-Server-15.04-or-newer).

# Settings and logs

Settings can be customized via the WebUI, WebAPI, or, if needed, manually editing the config file when qBitorrent is not running (`~/.config/qBittorrent/qBittorrent.conf).

The logs as well as other program data are located in `~/.local/share/data/qBittorrent`.

Consult the man page or pass `--help` for more information.