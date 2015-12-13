qBittorrent provides a search engine plugins management system. Thanks to this, you can ''easily'' write your own plugins to look for torrents in your favorite Bittorrent search engines and extend qBittorrent integrated search engine.

All you need is some motivation and some knowledge of [Python language](https://www.python.org).

But it would be also good idea to follow style guide:
[Python style guidelines](https://www.python.org/dev/peps/pep-0008/)

## Plugins specification

First, you must understand that a qBittorrent search engine plugin is actually a Python class file whose task is to contact a search engine website (e.g. [Mininova.org](http://www.mininova.org)), parse the results display by the web page and print them on stdout with the following syntax:

```
 link|name|size|seeds|leech|engine_url
```

One search result per line.

for example:

```
 http://www.mininova.org/get/1109509|ubuntu-hardy-desktop-i386-alpha-3 iso|711332986|0|0|http://www.mininova.org
 http://www.mininova.org/get/1088604|Xubuntu 7.10 for Virtual PC 2007|713901998|0|0|http://www.mininova.org
```

Note that the size is in provided bytes.

To achieve this task we provide several helper functions such as `prettyPrinter()`.

### prettyPrinter() helper function
In fact, you don't really need to pay attention to the output syntax because we provide a function for this called ''prettyPrinter(dictionary)''. You can import it using the following command:

```python
from novaprinter import prettyPrinter
```

You must pass to this function a dictionary containing the following keys (value should be -1 if you do not have the info):
* `link` => a string corresponding the the download link (points to the .torrent file)
* `name` => a unicode string corresponding to the torrent's name (i.e: "Ubuntu Linux v6.06")
* `size` => a string corresponding to the torrent size (i.e: "6 MB" or "200 KB" or "1.2 GB"...)
* `seeds` => the number of seeds for this torrent
* `leech` => the number of leechers for this torrent
* `engine_url` => the search engine url (i.e: http://www.mininova.org)

### retrieve_url() helper function

The ''retrieve_url()'' method takes an URL as parameter and returns the content of the URL as a string. This function is useful to get the search results from a Bittorrent search engine site. All you need to do is to pass the properly formatted URL to the function (the URL usually include GET parameters relative to search tokens, category, sorting, page number).

```python
from helpers import retrieve_url
dat = retrieve_url(self.url+'/search?q=%s&c=%s&o=52&p=%d'%(what, self.supported_categories[cat], i))
```

### download_file() helper function
The `download_file()` functions takes as a parameter the URL to a torrent file. This function will download the torrent to a temporary location and print on stdout:

```
 path_to_temporary_file url
```

It prints two values separated by a space:
* The path to the downloaded file (usually in /tmp folder)
* The URL from which the file was downloaded

Here is a usage example:

```python
from helpers import retrieve_url, download_file
print download_file(url)
> /tmp/esdzes http://www.mininova.org/get/123456
```

## Python Class File Structure

Your plugin should be named ''engine_name.py'', in lowercase and without spaces not any special characters. Here is the basic structure of this file:

```python
 #VERSION: 1.00
 #AUTHORS: YOUR_NAME (YOUR_MAIL)
 # LICENSING INFORMATION
 from novaprinter import prettyPrinter
 from helpers import retrieve_url, download_file
 import sgmllib
 
 # some other imports if necessary
 class engine_name(object):
   url = 'http://www.engine-url.org'
   name = 'Full engine name' # spaces and special characters are allowed here
   # Which search categories are supported by this search engine and their corresponding id
   # Possible categories are ('all', 'movies', 'tv', 'music', 'games', 'anime', 'software', 'pictures', 'books')
   supported_categories = {'all': '0', 'movies': '6', 'tv': '4', 'music': '1', 'games': '2', 'anime': '7', 'software': '3'}
 	
   def __init__(self):
     # some initialization
   
   def download_torrent(self, info):
     # Providing this function is optional. It can however be interesting to provide
     # your own torrent download implementation in case the search engine in question
     # does not allow traditional downloads (for example, cookie-based download).
     print download_file(info)
 	
   # DO NOT CHANGE the name and parameters of this function
   # This function will be the one called by nova2.py
   def search(self, what, cat='all'):
     # what is a string with the search tokens, already escaped (e.g. "Ubuntu+Linux")
     # cat is the name of a search category in ('all', 'movies', 'tv', 'music', 'games', 'anime', 'software', 'pictures', 'books')
     # 
     # Here you can do what you want to get the result from the
     # search engine website.
     # everytime you parse a result line, store it in a dictionary
     # and call the prettyPrint(your_dict) function
```

''PLEASE note that the filename (without .py extension) must be identical to the class name. Otherwise, qBittorrent will refuse to install it!''

## Parsing Results Web pages

After downloading the content of the web page containing the results (using `retrieve_url()`), you will want to parse it in order to create a `dict` per search result and call `prettyPrint(your_dict)` function to display it on stdout (in a format understandable by qBittorrent).

In order to parse the pages, you can use the following python modules (not exhaustive):
* [HTMLParser](https://docs.python.org/2/library/htmlparser.html) / [html.parser](https://docs.python.org/3/library/html.parser.html): Builtin python parser which replaces deprecated SGMLParser. Mostly similar to the SMGLParser ***[ADVISED METHOD]***
* `xml.dom.minidom`: xml parser. Be careful, this parser is very sensitive and the website must be fully XHTML compliant for this to work.
* `re`: If you like using regular expressions (''regex'')

## Examples

Do not hesitate to use the official search engine plugins as an example. They are available [here](https://github.com/qbittorrent/qBittorrent/tree/master/src/searchengine/nova/engines). 
* ''kickasstorrents.py'' uses ''json'' module
* ''torrentreactor.py'' uses ''HTMLParser'' module

## Test your plugin

It is better to test your plugin while developing it, before installing it in qBittorrent. Hence, we advise that you download [these files](https://github.com/qbittorrent/qBittorrent/tree/master/src/searchengine/nova). You will get the following structure:

```
 search_engine
 -> nova2.py # the main search engine script which calls the plugins
 -> nova2dl.py # standalone script called by qBittorrent to download a torrent using a particular search plugin
 -> helpers.py # contains helper functions you can use in your plugins such as retrieve_url() and download_file()
 -> novaprinter.py # contains some useful functions like prettyPrint(my_dict) to display your search results
 -> engines/ # folder that contains all the plugins (and possibily their favicons)
 ----> mininova.py
 ----> mininova.png
 ----> torrentreactor.py
 ----> torrentreactor.png
 ----> isohunt.py
 ----> isohunt.png
 ----> piratebay.py
 ----> piratebay.png
```

Put your plugin in `engines/` folder and then execute nova2.py script like this:

```
 ./nova2.py your_search_engine_name category search_tokens
 e.g.: ./nova2.py mininova all kubuntu linux
 e.g.: ./nova2.py btjunkie books ubuntu
```

## Install your plugin

To install your plugin in qBittorrent. Go to search tab in main window, click on ''Search engines...'' button. Then, a new window will pop up, containing the list of installed search engine plugins. Click on ''Install a new one'' at the bottom and select your `*.py` python script on you filesystem. If everything goes well, qBittorrent should notify you that it was successfully installed and your plugin should appear in the list.

## Post your working plugin

Once you managed to write a search engine plugin for qBittorrent that works, please post in on this wiki page [here](http://plugins.qbittorrent.org) so that the other users can use it too if they want. If you are lucky, your plugin may also be included in a future qBittorrent release.

## Notes

* As a convention, it is advised that you print the results sorted by number of seeds (the most seeds at the top) because these are usually the most interesting torrents.
* Please note that search engines usually display results on several pages. Hence, it is better to parse all these pages to get all results. All official plugins have multipage support.
* Some search engines do not provide all the informations required by `prettyPrinter()`. If it is the case, set -1 as value for the given key (i.e: "torrent_info['seeds'] = -1")
* Plugins packaged in a python are no longer directly installable since qBittorrent v2.0.0. You must provide qBittorrent with the python file.
