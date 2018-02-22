# ## **WIP**

From qBittorrent v4.0.5 and on, the WebUI architecture was expanded to allow the use of alternate set of files, allowing customization of the WebUI separated from the evolution of the core qBittorrent code.

Also, this mechanism is controlled via configuration options (via the core UI or config file), and the WebUI files (html, css, js, ...) are external to the program so it's easier than ever to test and modify the WebUI functionality, look and feel without need of rebuilding the project.

### Public and Private webUI files separation
In order to increase security, a `public` (login process handling) and `private` (WebUI functionality) separation of the files is implemented (see core WebUI files' segregation as ref: https://github.com/qbittorrent/qBittorrent/tree/master/src/webui/www)

### Config entries
* WebUI\AlternativeUIEnabled=`<true>/<false>`
* WebUI\RootFolder `<path>`

### First steps to an alternate WebUI:
Copy the files on https://github.com/qbittorrent/qBittorrent/tree/master/src/webui/www to a `<new folder>`, enable AltWebUI on the options and point the entry `WebUI\RootFolder` to `<new folder>`, then launch qBittorrent.
Access the WebUI as always, modify the files at `<new folder>` according to your needs and refresh the browser to see the changes reflected.

### References:
* Main WebUI PR: https://github.com/qbittorrent/qBittorrent/pull/7610

