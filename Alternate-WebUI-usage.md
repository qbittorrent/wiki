# Introduction

From qBittorrent v4.1.0 and on, the WebUI architecture was expanded to allow the use of alternate sets of WebUI sources, allowing customization of the WebUI and usage of community developed alternatives.

# Procedure

First, you need to download a compatible WebUI. You can find a list of unofficial community-developed WebUIs at [List of unofficial alternate WebUIs](https://github.com/qbittorrent/qBittorrent/wiki/List-of-known-alternate-WebUIs).

1. Under `Tools->Preferences->WebUI` enable `Use alternative WebUI`.
2. Choose a location that points to a WebUI-compatible folder location.
3. Restart qBittorrent or refresh your browser for changes to take effect.

You can also change these settings via the config file. The relevant entries are:

```
WebUI\AlternativeUIEnabled=true
WebUI\RootFolder=/path/to/some/alternate/webui
```
If you wish to customize the default one and maybe develop your own, refer to the article on [Developing alternate WebUIs](https://github.com/qbittorrent/qBittorrent/wiki/Developing-alternate-WebUIs-(WIP)).
