# ## **WIP**

From qBittorrent v4.0.5 and on, the webUI architecture was expanded to allow the use of alternate set of files, allowing customization of the webUI separated from the evolution of the core qBittorrent code.

Also, this mechanism is controlled via configuration options (via the core UI or config file), and the webUI files (html, css, js, ...) are external to the program so it's easier than ever to test and mofify the webUI functionality, look and feel without need of rebuilding the project.

### References:
* PR: https://github.com/qbittorrent/qBittorrent/pull/7610

