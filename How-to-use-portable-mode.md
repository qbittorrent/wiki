# Introduction

Portable mode makes it so that all qBittorrent config files get stored and read from one "portable" directory, to make it easy to migrate and share settings between qBittorrent installations. It also makes it possible to use qBittorrent without installing to- and reading configs from system directories. This feature is most useful for Windows users in general.

**NOTE: Only available for version 4.2.1 and above.**

# Linux

Use `--profile=path/to/config/dir --relative-fastresume`, where `path/to/config/dir` is the folder where you want your portable config to be stored or read from, if the config files already exist inside it.

Alternatively, create a folder literally named `profile` in the same directory as the qBittorrent executable, and just run the executable without the `--profile` and `--relative-fastresume` flags. The effect is the same as above, inside the `profile` directory. However, this is usually not what you want unless you are testing a build of qBittorrent you've compiled yourself in a directory that does not require elevated privileges.

**NOTE:** there are a bunch of caveats and pitfalls associated with portable mode and the `--relative-fastresume` flag in particular. Refer to the man page for more information and things to watch out for about these options. If just you want to use multiple configurations, use `--profile=path/to/config/dir` or `--configuration=config-name-here`. TODO: work on the new, complete man page is not yet finished.

# Windows

1. Place the qBittorrent binaries (`qbittorrent.exe` and `qbittorrent.pdb`) anywhere you want in your PC. You can extract them from the setup `.exe` file using 7-zip or similar (setup files are just fancy self-extracting archives).
2. Create a folder called `profile` in the same directory as the `qbittorrent.exe` and `qbittorrent.pdb` files.
3. Execute `qbittorrent.exe`. It will initialize the needed configuration files within the `profile` folder and use them during execution.

- Note: You can copy qBittorrent configuration files from any installation, portable or not, to the matching directories inside the `profile` directory. When started, qBittorrent will use those files if they exist. You can of course move the `profile` directory around including to different machines, and any `qbittorrent.exe` started beside it will use those configs. Beware that some settings that deal with specific paths may not work out of the box when migrating, because paths may not be exactly the same and may not exist across machines.
