# ## **WIP**

From qBittorrent v4.0.5 and on, the webUI architecture was expanded to allow the use of alternate set of files, allowing customization of the webUI separated from the evolution of the core qBittorrent code.

Also, this mechanism is controlled via configuration options (via the core UI or config file), and the webUI files (html, css, js, ...) are external to the program so it's easier than ever to test and mofify the webUI functionality, look and feel without need of rebuilding the project.

--- private and public files separation

--- config entries
WebUI/AlternativeUIEnabled=`<true>/<false>`
WebUI/RootFolder `path`

### First steps to an alternate webUI:
Copy the files on XYZ to a `new folder`, enable altwebUI on the options and point the entry ABC to `new folder`, then launch qBittorrent.
Access the webUI as always, modify the files at `new folder` according to your needs and refresh the browser to see the changes reflected.

### References:
* Main webUI PR: https://github.com/qbittorrent/qBittorrent/pull/7610

