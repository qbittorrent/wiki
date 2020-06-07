This WebUI API documentation applies to qBittorrent v3.1.x. For other API versions, visit [WebUI API](https://github.com/qbittorrent/qBittorrent/wiki#WebUI-API).

# Table of Contents #

1. [Authorization](#authorization)
1. [GET methods](#get-methods)
  1. [Shutdown qBittorrent](#shutdown-qbittorrent)
  1. [Get torrent list](#get-torrent-list)
  1. [Get torrent generic properties](#get-torrent-generic-properties)
  1. [Get torrent trackers](#get-torrent-trackers)
  1. [Get torrent contents](#get-torrent-contents)
  1. [Get global transfer info](#get-global-transfer-info)
  1. [Get qBittorrent preferences](#get-qbittorrent-preferences)
1. [POST methods](#post-methods)
  1. [Download torrent from URL](#download-torrent-from-url)
  1. [Upload torrent from disk](#upload-torrent-from-disk)
  1. [Add trackers to torrent](#add-trackers-to-torrent)
  1. [Pause torrent](#pause-torrent)
  1. [Pause all torrents](#pause-all-torrents)
  1. [Resume torrent](#resume-torrent)
  1. [Resume all torrents](#resume-all-torrents)
  1. [Delete torrent](#delete-torrent)
  1. [Delete torrent with downloaded data](#delete-torrent-with-downloaded-data)
  1. [Recheck torrent](#recheck-torrent)
  1. [Increase torrent priority](#increase-torrent-priority)
  1. [Decrease torrent priority](#decrease-torrent-priority)
  1. [Maximal torrent priority](#maximal-torrent-priority)
  1. [Minimal torrent priority](#minimal-torrent-priority)
  1. [Set file priority](#set-file-priority)
  1. [Get global download limit](#get-global-download-limit)
  1. [Set global download limit](#set-global-download-limit)
  1. [Get global upload limit](#get-global-upload-limit)
  1. [Set global upload limit](#set-global-upload-limit)
  1. [Get torrent download limit](#get-torrent-download-limit)
  1. [Set torrent download limit](#set-torrent-download-limit)
  1. [Get torrent upload limit](#get-torrent-upload-limit)
  1. [Set torrent upload limit](#set-torrent-upload-limit)
  1. [Set qBittorrent preferences](#set-qbittorrent-preferences)
1. [Additional information](#additional-information)
  1. [Version 3.0.8 bugs](#version-308-bugs)

***

# Authorization #

Authorization requires using `Authorization` header inside GET/POST requests. qBittorrent uses the standard Digest Authorization type (using a MD5 hash generator).

For example on Python using [requests](http://docs.python-requests.org/en/latest/):

    import requests
    from requests.auth import HTTPDigestAuth

    response = requests.post('http://127.0.0.1:8080/command/download', {'urls': magnet_link}, auth=HTTPDigestAuth(username, password))
    if not response.ok:
        response.raise_for_status()
    response.content

1. Digest Authorization standard

  [This page](http://www.w3.org/People/Raggett/security/draft-ietf-http-digest-aa-00.txt) describes how this auth method should ideally work. The most important thing there is `response` field generation.
  Response is generated like this: `MD5 ( A1 + ':' + nonce + ':' + A2 )` where <br/>
  A1 is `MD5 (username + ':' + realm + ':' + password)` <br/>
  A2 is `MD5 (<method type string (e.g. GET or POST)> + ':' + uri)`

1. Server Auth response

  When you fail Authorization or don't supply required header qBittorrent will send the following reply (example):

  ```http
HTTP/1.1 401 Unauthorized
WWW-Authenticate: Digest realm="Web UI Access", nonce="a3f396f2dcc1cafae73637e2ac321134", opaque="a3f396f2dcc1cafae73637e2ac321134", stale="false", algorithm="MD5", qop="auth"
  ```

  `nonce` and `opaque` values will be random, nevertheless it looks like matching them is not required for successful authorization - qBittorrent doesn't track user session.

1. Client Auth request

  We will now generate our request to the server containing proper Authorization header based on the example above.

  1. User-Agent

      You can place whatever you want as user agent, it is not required anyway.<br/>
      `User-Agent: Fiddler`

  1. Host

      Server address or domain name.<br/>
      `Host: 127.0.0.1`

  1. Authorization

      This header is required.<br/>

      ```
Authorization: Digest username="admin", realm="Web UI Access", nonce="a3f396f2dcc1cafae73637e2ac321134", uri="/", response="4067cfe4c029cd00b56076c78abd034c"
      ```

      All fields in this header are required. Since `nonce` is not tracked by qBittorrent you should MD5-generate it yourself either one-time or each time you do any request. `uri` is not checked, if you want to GET host/someurl and `uri` doesn't match this value - no problem, just remember that `uri` is used in `response` generation.

      In Short: **You don't need a previous server reply in order to generate a proper request.**

      You must supply Authorization header in any POST/GET request you issue.

  1. End Result

      Server reply:
      
      ```http
HTTP/1.1 401 Unauthorized
WWW-Authenticate: Digest realm="Web UI Access", nonce="a3f396f2dcc1cafae73637e2ac321134", opaque="a3f396f2dcc1cafae73637e2ac321134", stale="false", algorithm="MD5", qop="auth"
      ```

      Client request:

      ```http
GET / HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Authorization: Digest username="admin", realm="Web UI Access", nonce="a3f396f2dcc1cafae73637e2ac321134", uri="/", response="4067cfe4c029cd00b56076c78abd034c"
      ```

# GET methods #

### Shutdown qBittorrent ###

```http
GET /command/shutdown HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Authorization: your_auth_string
```

### Get torrent list ###

```http
GET /json/torrents HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Authorization: your_auth_string
```
Server will return the following reply (example):

```http
HTTP/1.1 200 OK
content-type: text/javascript
content-length: length

[{"hash":"0283b35cc14387a4f6d01de33aa012fb1b740df6","name":"Sword of the Stars II Enhanced Edition","size":"1.8 ГиБ","progress":1,"dlspeed":"0 Б/с","upspeed":"0 Б/с","priority":"*","num_seeds":"0","num_leechs":"0","ratio":"0.1","eta":"∞","state":"stalledUP"},{another_torrent_info}]
```
where

`hash` - torrent hash; most queries will use torrent hash as parameter<br/>
`name` - torrent name<br/>
`size` - size of files and folders in torrent selected for download; possible values:<br/>

1. `Unknown` - this may happen, which means qBt couldn't determine torrent size for some reason
1. `X suffix` - X B/KiB/MiB/GiB/TiB

  Proper suffix is appended automatically by qBt.

`progress` - float number for download completion percentage, where 1 == 100%, 0 == 0%, and, 0.58 == 58%<br/>
`dlspeed` and `upspeed`: torrent download and upload speed respectively, possible values:<br/>

1. `Unknown` - qBt couldn't determine torrent speed for some reason
1. `X suffix/s` - X B/KiB/MiB/GiB/TiB

`priority` - torrent number in priority queue; contains `*` if queuing is disabled or torrent is in seed mode<br/>
`num_seeds` and `num_leechs` - number of peers and lecchers<br/>
`ratio` - Uploaded/Downloaded ratio, rounded to first digit after comma; contains `∞` if ratio > 100<br/>
`eta` - contains `∞` if torrent is seeding only or eta >= 8640000 seconds; possible values:<br/>

1. `0` - zero
1. `< 1m` - less than a minute
1. `MMm` - MM minutes
1. `HHh MMm` - HH hours and MM minutes
1. `DDd HHh` - DD days and HH hours

  DD/HH/MM values can be truncated if first digit is zero

`state` - torrent state, possible values:<br/>

1. `error` - some error occurred, applies to paused torrents
1. `pausedUP` - torrent is paused and has finished downloading
1. `pausedDL` - torrent is paused and has **NOT** finished downloading
1. `queuedUP` - queuing is enabled and torrent is queued for upload
1. `queuedDL` - queuing is enabled and torrent is queued for download
1. `uploading` - torrent is being seeded and data is being transfered
1. `stalledUP` - torrent is being seeded, but no connection were made
1. `checkingUP` - torrent has finished downloading and is being checked; this status also applies to preallocation (if enabled) and checking resume data on qBt startup
1. `checkingDL` - same as `checkingUP`, but torrent has **NOT** finished downloading
1. `downloading` - torrent is being downloaded and data is being transfered
1. `stalledDL` - torrent is being downloaded, but no connection were made

**BIG FAT WARNING:** `size`, `dlspeed`, `upspeed` and `eta` suffixes (e.g. MiB, KiB/s) will depend on current language selected in qBt; these suffixes will be translated.

**BIG FAT WARNING 2:** Raw data this methods provides can exceed 40 KiB for ~200 torrents. Continuous polling is strongly discouraged for mobile clients.

### Get torrent generic properties ###

Requires known torrent hash, get 'em from [torrent list](#torrentlist).

```http
GET /json/propertiesGeneral/fae6e49afa359ab07c3ef169438ff95d870bc178 HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Authorization: your_auth_string
```

If your torrent hash is invalid server will reply with:

```http
HTTP/1.1 200 OK
content-type: text/javascript
content-length: 0
```

Otherwise server will return the following reply (example):

```http
HTTP/1.1 200 OK
content-type: text/javascript
content-length: length

{"save_path":"D:/Downloads/somefolder","creation_date":"16 ноября 2011 г. 20:52:54","piece_size":"4.0 МиБ","comment":"comment_string_if_any","total_wasted":"0 Б","total_uploaded":"41.9 ГиБ (0 Б за эту сессию)","total_downloaded":"261.2 МиБ (0 Б за эту сессию)","up_limit":"∞","dl_limit":"∞","time_elapsed":"53д 0ч (Раздается 0)","nb_connections":"0 (100 макс)","share_ratio":"∞"}
```
where

`path` - path where torrent contents are saved, separated by slashes<br/>
`creation_date` - (translated string) date when torrent was added<br/>
`piece_size` - (translated string) torrent piece size<br/>
`comment` - torrent comment<br/>
`total_wasted` - (translated string) amount of data 'wasted'<br/>
`total_uploaded` and `total_downloaded` - (translated string) amounts of data uploaded and downloaded, value in parentheses count current session data only<br/>
`up_limit` and `dl_limit` - (translated string) upload and download speed limits for current torrent<br/>
`time_elapsed` - (translated string) total time active; value in parentheses represents current seeding time<br/>
`nb_connections` - (translated string) number of connections, value in parentheses represents maximum number of connections per torrent set in preferences<br/>
`share_ratio` - (translated string) UL/DL ratio; contains `∞` for ratios > 100 <br/>

### Get torrent trackers ###

Requires known torrent hash, get 'em from [torrent list](#torrentlist).

```http
GET http://127.0.0.1/json/propertiesTrackers/fae6e49afa359ab07c3ef169438ff95d870bc178 HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Authorization: your_auth_string
```

If your torrent hash is invalid server will reply with:

```http
HTTP/1.1 200 OK
content-type: text/javascript
content-length: 2

[]
```

Otherwise server will return the following reply (example):

```http
HTTP/1.1 200 OK
content-type: text/javascript
content-length: length

[{"url":"http://someaddress/announce","status":"Работает","num_peers":"1","msg":""},{"url":"http://retracker.local/announce","status":"Не соединился","num_peers":"0","msg":""}]
```
where

`url` - tracker url<br/>
`status` - (translated string) tracker status; possible values:<br/>

  1. `Working` - tracker has been contacted and is working
  1. `Updating...` - tracker is currently being updated
  1. `Not working` - tracker has been contacted, but it is not working (or doesn't send proper replies)
  1. `Not contacted yet` - tracker has not been contacted yet
  
`num_peers` - number of peers for current torrent eported by tracker<br/>
`msg` - tracker message (there is no way of knowing what this message is - it's up to tracker admins)<br/>

### Get torrent contents ###

Requires known torrent hash, get 'em from [torrent list](#torrentlist).

```http
GET http://127.0.0.1/json/propertiesFiles/fae6e49afa359ab07c3ef169438ff95d870bc178 HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Authorization: your_auth_string
```

If your torrent hash is invalid server will reply with:

```http
HTTP/1.1 200 OK
content-type: text/javascript
content-length: 0
```

Otherwise server will return the following reply (example):

```http
HTTP/1.1 200 OK
content-type: text/javascript
content-length: length

[{"name":"Isekai no Seikishi Monogatari 01.mka","size":"70.4 МиБ","progress":0,"priority":0,"is_seed":true},{"name":"Isekai no Seikishi Monogatari 02.mka","size":"62.4 МиБ","progress":0,"priority":0}]
```
where

`name` - file name<br/>
`size` - (translated string) file size<br/>
`progress` - float value, indicating file progress; 0 == 0% and 1 == 100%<br/>
`priority` - file priority, possible values:<br/>

  1. `0` - do not download
  1. `1` - normal priority
  1. `2` - high priority
  1. `7` - maximal priority

`is_seed` - only present for the first file in torrent; true if torrent is in seed mode<br/>

### Get global transfer info ###

This method returns info you usually see in qBt status bar.

```http
GET http://127.0.0.1/json/transferInfo HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Authorization: your_auth_string
```

Server reply (example):

```http
HTTP/1.1 200 OK
content-type: text/javascript
content-length: length

{"dl_info":"Приём: 0 Б/с - Передано: 0 Б","up_info":"Отдача: 209.9 КиБ/с - Передано: 7.2 ГиБ"}
```
where

`dl_info` - (translated string) contains current global downalod speed and global amount of data downaloded during this session<br/>
`up_info` - (translated string) contains current global upload speed and global amount of data uploaded during this session<br/>

### Get qBittorrent preferences ###

```http
GET http://127.0.0.1/json/preferences HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Authorization: your_auth_string
```

Server reply; contents may vary depending on which settings are present in qBittorrent.ini (example):

```http
HTTP/1.1 200 OK
content-type: text/javascript
content-length: length

{"locale":"ru_RU","save_path":"C:/Users/Dayman/Downloads","temp_path_enabled":false,"temp_path":"C:/Users/Dayman/Documents/Downloads/temp","scan_dirs":["D:/Browser Downloads"],"download_in_scan_dirs":[false],"export_dir_enabled":false,"export_dir":"","mail_notification_enabled":false,"mail_notification_email":"","mail_notification_smtp":"smtp.changeme.com","mail_notification_ssl_enabled":false,"mail_notification_auth_enabled":false,"mail_notification_username":"","mail_notification_password":"","autorun_enabled":false,"autorun_program":"","preallocate_all":false,"queueing_enabled":true,"max_active_downloads":2,"max_active_torrents":200,"max_active_uploads":200,"dont_count_slow_torrents":false,"incomplete_files_ext":false,"listen_port":31498,"upnp":false,"dl_limit":3072,"up_limit":3072,"max_connec":500,"max_connec_per_torrent":100,"max_uploads_per_torrent":15,"enable_utp":true,"limit_utp_rate":false,"limit_tcp_overhead":true,"alt_dl_limit":1024,"alt_up_limit":2048,"scheduler_enabled":false,"schedule_from_hour":8,"schedule_from_min":0,"schedule_to_hour":20,"schedule_to_min":0,"scheduler_days":0,"dht":true,"dhtSameAsBT":true,"dht_port":6881,"pex":true,"lsd":true,"encryption":0,"anonymous_mode":false,"proxy_type":-1,"proxy_ip":"0.0.0.0","proxy_port":8080,"proxy_peer_connections":false,"proxy_auth_enabled":false,"proxy_username":"","proxy_password":"","ip_filter_enabled":false,"ip_filter_path":null,"web_ui_port":80,"web_ui_username":"admin","web_ui_password":"8888efb275743684292cff99f57867a9","bypass_local_auth":false,"use_https":false,"ssl_key":"","ssl_cert":"","dyndns_enabled":false,"dyndns_service":0,"dyndns_username":"","dyndns_password":"","dyndns_domain":"changeme.dyndns.org"}
```
where

`locale` - currently selected language (e.g. en_GB for english)<br/>
`save_path` - default save path for torrents, separated by slashes<br/>
`temp_path_enabled` - true if folder for incomplete torrents is enabled<br/>
`temp_path` - path for incomplete torrents, separated by slashes<br/>
`scan_dirs` - list of watch folders to add torrent automatically; slashes are used as path separators; list entries are separated by commas<br/>
`download_in_scan_dirs` - true if torrents should be downloaded to watch folder; list entries are separated by commas<br/>
`export_dir_enabled` - true if .torrent file should be copied to export directory upon adding<br/>
`export_dir` - path to directory to copy .torrent files if `export_dir_enabled` is enabled; path is separated by slashes<br/>
`mail_notification_enabled` - true if e-mail notification should be enabled<br/>
`mail_notification_email` - e-mail to send notifications to<br/>
`mail_notification_smtp` - smtp server for e-mail notifications<br/>
`mail_notification_ssl_enabled` - true if smtp server requires SSL connection<br/>
`mail_notification_auth_enabled` - true if smtp server requires authentication<br/>
`mail_notification_username` - username for smtp authentication<br/>
`mail_notification_password` - password for smtp authentication<br/>
`autorun_enabled` - true if external program should be run after torrent has finished downloading<br/>
`autorun_program` - program path/name/arguments to run if `autorun_enabled` is enabled; path is separated by slashes; you can use `%f` and `%n` arguments, which will be expanded by qBittorent as path_to_torrent_file and torrent_name (from the GUI; not the .torrent file name) respectively<br/>
`preallocate_all` - true if file preallocation should take place, otherwise sparse files are used<br/>
`queueing_enabled` - true if torrent queuing is enabled<br/>
`max_active_downloads` - maximum number of active simultaneous downloads<br/>
`max_active_torrents` - maximum number of active simultaneous downloads and uploads<br/>
`max_active_uploads` - maximum number of active simultaneous uploads<br/>
`dont_count_slow_torrents` - if true torrents w/o any activity (stalled ones) will not be counted towards `max_active_*` limits; see [dont_count_slow_torrents](https://www.libtorrent.org/reference-Settings.html#dont_count_slow_torrents) for more information<br/>
`incomplete_files_ext` - if true `.!qB` extension will be appended to incomplete files<br/>
`listen_port` - port for incoming connections<br/>
`upnp` - true if UPnP/NAT-PMP is enabled<br/>
`dl_limit` - global download speed limit in KiB/s; `-1` means no limit is applied<br/>
`up_limit` - global upload speed limit in KiB/s; `-1` means no limit is applied<br/>
`max_connec` - maximum global number of simultaneous connections<br/>
`max_connec_per_torrent` - maximum number of simultaneous connections per torrent<br/>
`max_uploads_per_torrent` - maximum number of upload slots per torrent<br/>
`enable_utp` - true if uTP protocol should be enabled; this option is only available in qBittorent built against libtorrent version 0.16.X and higher<br/>
`limit_utp_rate` - true if `[du]l_limit` should be applied to uTP connections; this option is only available in qBittorent built against libtorrent version 0.16.X and higher<br/>
`limit_tcp_overhead` - true if `[du]l_limit` should be applied to estimated TCP overhead (service data: e.g. packet headers)<br/>
`alt_dl_limit` - alternative global download speed limit in KiB/s<br/>
`alt_up_limit` - alternative global upload speed limit in KiB/s<br/>
`scheduler_enabled` - true if alternative limits should be applied according to schedule<br/>
`schedule_from_hour` - scheduler starting hour<br/>
`schedule_from_min` - scheduler starting minute<br/>
`schedule_to_hour` - scheduler ending hour<br/>
`schedule_to_min` - scheduler ending minute<br/>
`scheduler_days` - scheduler days; possible values:<br/>

  1. `0` - every day
  1. `1` - every weekday
  1. `2` - every weekend
  1. `3` - every Monday
  1. `4` - every Tuesday
  1. `5` - every Wednesday
  1. `6` - every Thursday
  1. `7` - every Friday
  1. `8` - every Saturday
  1. `9` - every Sunday

`dht` - true if DHT is enabled<br/>
`dhtSameAsBT` - true if DHT port should match TCP port<br/>
`dht_port` - DHT port if `dhtSameAsBT` is false<br/>
`pex` - true if PeX is enabled<br/>
`lsd` - true if LSD is eanbled<br/>
`encryption` - possible values:<br/>

  1. `0` - prefer encryption
  1. `1` - force encryption on
  1. `2` - force encryption off

    First options allows you to use both encrypted and unencrypted connections (this is the default); other options are mutually exclusive: e.g. by forcing encryption on you won't be able to use unencrypted connections and vice versa.

`anonymous_mode` - if true anonymous mode will be enabled; read more [here](Anonymous-Mode); this option is only available in qBittorent built against libtorrent version 0.16.X and higher<br/>
`proxy_type` - possible values:<br/>

  1. `-1` - proxy is disabled
  1. `1` - HTTP proxy without authentication
  1. `2` - SOCKS5 proxy without authentication
  1. `3` - HTTP proxy with authentication
  1. `4` - SOCKS5 proxy with authentication
  1. `5` - SOCKS4 proxy without authentication

`proxy_ip` - proxy IP address or domain name<br/>
`proxy_port` - proxy port<br/>
`proxy_peer_connections` - true if peer and web seed connections should be proxified; this option will have any effect only in qBittorent built against libtorrent version 0.16.X and higher<br/>
`proxy_auth_enabled` - true proxy requires authentication; doesn't apply to SOCKS4 proxies<br/>
`proxy_username` - username for proxy authentication<br/>
`proxy_password` - password for proxy authentication<br/>
`ip_filter_enabled` - true if external IP filter should be enabled<br/>
`ip_filter_path` - path to IP filter file (.dat, .p2p, .p2b files are supported); path is separated by slashes<br/>
`web_ui_port` - WebUI port<br/>
`web_ui_username` - WebUI username<br/>
`web_ui_password` - MD5 hash of WebUI password; hash is generated from the following string: `username:Web UI Access:plain_text_web_ui_password`<br/>
`bypass_local_auth` - true if auithetication challenge for loopback address (127.0.0.1) should be disabled<br/>
`use_https` - true if WebUI HTTPS access is eanbled<br/>
`ssl_key` - SSL keyfile contents (this is a not a path)<br/>
`ssl_cert` - SSL certificate contents (this is a not a path)<br/>
`dyndns_enabled` - true if server DNS should be updated dynamically<br/>
`dyndns_service` - possible values:<br/>

  1. `0` - use DyDNS
  1. `1` - use NOIP

`dyndns_username` - username for DDNS service<br/>
`dyndns_password` - password for DDNS service<br/>
`dyndns_domain` - your DDNS domain name <br/>

# POST methods #

**Please adjust you auth string aacordingly for POST methods.**

### Download torrent from URL ###

This method can add torrents from urls and magnet links. BC links are also supported.

```http
POST http://127.0.0.1/command/download HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Authorization: your_auth_string
Content-Type: application/x-www-form-urlencoded
Content-Length: length

urls=http://www.nyaa.eu/?page=download%26tid=305093%0Ahttp://www.nyaa.eu/?page=download%26tid=305255%0Amagnet:?xt=urn:btih:4c284ebef5bf0d967e2e174cfe825d9fb40ae5e1%26dn=QBittorrent+2.8.4+Win7+Vista+64+working+version%26tr=udp%3A%2F%2Ftracker.openbittorrent.com%3A80%26tr=udp%3A%2F%2Ftracker.publicbt.com%3A80%26tr=udp%3A%2F%2Ftracker.istole.it%3A6969%26tr=udp%3A%2F%2Ftracker.ccc.de%3A80
```

Please note that:

1. `Content-Type: application/x-www-form-urlencoded` is required
1. `urls` contains a list of links; links are separated with `%0A` (LF newline)
1. `http://`, `https://`, `magnet:` and `bc://bt/` links are supported
1. Links' contents must be escaped, e.g. `&` becomes `%26` (don't know about other characters but ampersand **MUST** be escaped)

No matter if successful or not server will return the following reply:

```http
HTTP/1.1 200 OK
```

### Upload torrent from disk ###

```http
POST http://127.0.0.1/command/upload HTTP/1.1
Content-Type: multipart/form-data; boundary=-------------------------acebdf13572468
User-Agent: Fiddler
Host: 127.0.0.1
Authorization: your_auth_string
Content-Length: length

---------------------------acebdf13572468
Content-Disposition: form-data; name="torrents"; filename="8f18036b7a205c9347cb84a253975e12f7adddf2.torrent"
Content-Type: application/x-bittorrent

file_binary_data_goes_here
---------------------------acebdf13572468
Content-Disposition: form-data; name="torrents"; filename="UFS.torrent"
Content-Type: application/x-bittorrent

file_binary_data_goes_here
---------------------------acebdf13572468--

```

The above example will add two torrent files. `file_binary_data_goes_here` represents raw data of torrent file (basically a byte array).

No matter if successful or not server will return the following reply:

```http
HTTP/1.1 200 OK
content-type: text/html
content-length: 64

<script type="text/javascript">window.parent.hideAll();</script>HTTP/1.1 200 OK
content-type: text/html
content-length: 64

<script type="text/javascript">window.parent.hideAll();</script>
```

### Add trackers to torrent ###

Requires known torrent hash, get 'em from [torrent list](#torrentlist).

```http
POST http://127.0.0.1/command/addTrackers HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Authorization: your_auth_string
Content-Type: application/x-www-form-urlencoded
Content-Length: length

hash=fae6e49afa359ab07c3ef169438ff95d870bc178&urls=http://192.168.0.1/announce%0Audp://192.168.0.1:3333/dummyAnnounce
```

This adds two trackers to torrent with hash `fae6e49afa359ab07c3ef169438ff95d870bc178`. Note `%0A` (aka LF newline) between trackers. Ampersand in tracker urls **MUST** be escaped.

No matter if successful or not server will return the following reply:

```http
HTTP/1.1 200 OK
```

### Pause torrent ###

Requires known torrent hash, get 'em from [torrent list](#torrentlist).

```http
POST http://127.0.0.1/command/pause HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Authorization: your_auth_string
Content-Type: application/x-www-form-urlencoded
Content-Length: length

hash=fae6e49afa359ab07c3ef169438ff95d870bc178
```

No matter if successful or not server will return the following reply:

```http
HTTP/1.1 200 OK
```

### Pause all torrents ###

```http
POST http://127.0.0.1/command/pauseall HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Authorization: your_auth_string
Content-Length: 0
```

No matter if successful or not server will return the following reply:

```http
HTTP/1.1 200 OK
```

### Resume torrent ###

Requires known torrent hash, get 'em from [torrent list](#torrentlist).

```http
POST http://127.0.0.1/command/resume HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Authorization: your_auth_string
Content-Type: application/x-www-form-urlencoded
Content-Length: length

hash=fae6e49afa359ab07c3ef169438ff95d870bc178
```

No matter if successful or not server will return the following reply:

```http
HTTP/1.1 200 OK
```

### Resume all torrents ###

```http
POST http://127.0.0.1/command/resumeall HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Authorization: your_auth_string
Content-Length: 0
```

No matter if successful or not server will return the following reply:

```http
HTTP/1.1 200 OK
```

### Delete torrent ###

Requires known torrent hash, get 'em from [torrent list](#torrentlist).

```http
POST http://127.0.0.1/command/delete HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Authorization: your_auth_string
Content-Type: application/x-www-form-urlencoded
Content-Length: length

hashes=fae6e49afa359ab07c3ef169438ff95d870bc178
```

`hashes` can contain multiple hashes separated by `|`

No matter if successful or not server will return the following reply:

```http
HTTP/1.1 200 OK
```

### Delete torrent with downloaded data ###

Requires known torrent hash, get 'em from [torrent list](#torrentlist).

```http
POST http://127.0.0.1/command/deletePerm HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Authorization: your_auth_string
Content-Type: application/x-www-form-urlencoded
Content-Length: length

hashes=fae6e49afa359ab07c3ef169438ff95d870bc178
```

`hashes` can contain multiple hashes separated by `|`

No matter if successful or not server will return the following reply:

```http
HTTP/1.1 200 OK
```

### Recheck torrent ###

Requires known torrent hash, get 'em from [torrent list](#torrentlist).

```http
POST http://127.0.0.1/command/recheck HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Authorization: your_auth_string
Content-Type: application/x-www-form-urlencoded
Content-Length: length

hash=fae6e49afa359ab07c3ef169438ff95d870bc178
```

No matter if successful or not server will return the following reply:

```http
HTTP/1.1 200 OK
```
### Increase torrent priority ###

Requires known torrent hash, get 'em from [torrent list](#torrentlist).

```http
POST http://127.0.0.1/command/increasePrio HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Authorization: your_auth_string
Content-Type: application/x-www-form-urlencoded
Content-Length: length

hashes=fae6e49afa359ab07c3ef169438ff95d870bc178
```

`hashes` can contain multiple hashes separated by `|`

No matter if successful or not server will return the following reply:

```http
HTTP/1.1 200 OK
```

### Decrease torrent priority ###

Requires known torrent hash, get 'em from [torrent list](#torrentlist).

```http
POST http://127.0.0.1/command/decreasePrio HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Authorization: your_auth_string
Content-Type: application/x-www-form-urlencoded
Content-Length: length

hashes=fae6e49afa359ab07c3ef169438ff95d870bc178
```

`hashes` can contain multiple hashes separated by `|`

No matter if successful or not server will return the following reply:

```http
HTTP/1.1 200 OK
```

### Maximal torrent priority ###

Requires known torrent hash, get 'em from [torrent list](#torrentlist).

```http
POST http://127.0.0.1/command/topPrio HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Authorization: your_auth_string
Content-Type: application/x-www-form-urlencoded
Content-Length: length

hashes=fae6e49afa359ab07c3ef169438ff95d870bc178
```

`hashes` can contain multiple hashes separated by `|`

No matter if successful or not server will return the following reply:

```http
HTTP/1.1 200 OK
```

### Minimal torrent priority ###

Requires known torrent hash, get 'em from [torrent list](#torrentlist).

```http
POST http://127.0.0.1/command/bottomPrio HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Authorization: your_auth_string
Content-Type: application/x-www-form-urlencoded
Content-Length: length

hashes=fae6e49afa359ab07c3ef169438ff95d870bc178
```

`hashes` can contain multiple hashes separated by `|`

No matter if successful or not server will return the following reply:

```http
HTTP/1.1 200 OK
```

### Set file priority ###

Requires known torrent hash, get 'em from [torrent list](#torrentlist).

```http
POST http://127.0.0.1/command/setFilePrio HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Authorization: your_auth_string
Content-Type: application/x-www-form-urlencoded
Content-Length: length

hash=fae6e49afa359ab07c3ef169438ff95d870bc178&id=0&priority=7
```

Please consult [torrent contents API](#propscont) for possible `priority` values. `id` values coresspond to contents returned by [torrent contents API](#propscont), e.g. `id=0` for first file, `id=1` for second file, etc.

No matter if successful or not server will return the following reply:

```http
HTTP/1.1 200 OK
```

### Get global download limit ###

```http
POST http://127.0.0.1/command/getGlobalDlLimit HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Authorization: your_auth_string
Content-Length: 0
```

Server reply (example):

```http
HTTP/1.1 200 OK
content-type: text/html
content-length: length

3145728
```

`3145728` is the value of current global download speed limit in bytes; this value will be zero if no limit is applied.

### Set global download limit ###

```http
POST http://127.0.0.1/command/setGlobalDlLimit HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Authorization: your_auth_string
Content-Type: application/x-www-form-urlencoded
Content-Length: length

limit=4194304
```

`limit` is global download speed limit you want to set in bytes.

No matter if successful or not server will return the following reply:

```http
HTTP/1.1 200 OK
```

### Get global upload limit ###

```http
POST http://127.0.0.1/command/getGlobalUpLimit HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Authorization: your_auth_string
Content-Length: 0
```

Server reply (example):

```http
HTTP/1.1 200 OK
content-type: text/html
content-length: length

3145728
```

`3145728` is the value of current global upload speed limit in bytes; this value will be zero if no limit is applied.

### Set global upload limit ###

```http
POST http://127.0.0.1/command/setGlobalUpLimit HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Authorization: your_auth_string
Content-Type: application/x-www-form-urlencoded
Content-Length: length

limit=4194304
```

`limit` is global upload speed limit you want to set in bytes.

No matter if successful or not server will return the following reply:

```http
HTTP/1.1 200 OK
```

### Get torrent download limit ###

Requires known torrent hash, get 'em from [torrent list](#torrentlist).

```http
POST http://127.0.0.1/command/getTorrentDlLimit HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Authorization: your_auth_string
Content-Type: application/x-www-form-urlencoded
Content-Length: length

hash=fae6e49afa359ab07c3ef169438ff95d870bc178
```

Server reply (example):

```http
HTTP/1.1 200 OK
content-type: text/html
content-length: length

338944
```

`338944` is the value of current torrent download speed limit in bytes; this value will be zero if no limit is applied.

### Set torrent download limit ###

Requires known torrent hash, get 'em from [torrent list](#torrentlist).

```http
POST http://127.0.0.1/command/setTorrentDlLimit HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Authorization: your_auth_string
Content-Type: application/x-www-form-urlencoded
Content-Length: length

hash=fae6e49afa359ab07c3ef169438ff95d870bc178&limit=131072
```

`limit` is torrent download speed limit you want to set in bytes.

No matter if successful or not server will return the following reply:

```http
HTTP/1.1 200 OK
```

### Get torrent upload limit ###

Requires known torrent hash, get 'em from [torrent list](#torrentlist).

```http
POST http://127.0.0.1/command/getTorrentUpLimit HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Authorization: your_auth_string
Content-Type: application/x-www-form-urlencoded
Content-Length: length

hash=fae6e49afa359ab07c3ef169438ff95d870bc178
```

Server reply (example):

```http
HTTP/1.1 200 OK
content-type: text/html
content-length: length

338944
```

`338944` is the value of current torrent upload speed limit in bytes; this value will be zero if no limit is applied.

### Set torrent upload limit ###

Requires known torrent hash, get 'em from [torrent list](#torrentlist).

```http
POST http://127.0.0.1/command/setTorrentUpLimit HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Authorization: your_auth_string
Content-Type: application/x-www-form-urlencoded
Content-Length: length

hash=fae6e49afa359ab07c3ef169438ff95d870bc178&limit=131072
```

`limit` is torrent download speed limit you want to set in bytes.

No matter if successful or not server will return the following reply:

```http
HTTP/1.1 200 OK
```

### Set qBittorrent preferences ###

```http
POST http://127.0.0.1/command/setPreferences HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Authorization: your_auth_string
Content-Type: application/x-www-form-urlencoded
Content-Length: length

json={"save_path":"C:/Users/Dayman/Downloads","queueing_enabled":false,"scan_dirs":["C:/Games","D:/Downloads"],"download_in_scan_dirs":[false,true]}
```

No matter if successful or not server will return the following reply:

```http
HTTP/1.1 200 OK
```

  1. There is no need to pass all possible preferences' `token:value` pairs if you only want to change one option
  1. When setting preferences `scan_dirs` must **always** be accompanied with `download_in_scan_dirs`
  1. Paths in `scan_dirs` must exist, otherwise this option will have no effect
  1. String values must be quoted; integer and boolean values must never be quoted

For a list of possible preference options see [Get qBittorrent preferences](#prefget)

# Additional information #

### Version 3.0.8 bugs ###

The following WebUI-related bugs exist in qBittorent v3.0.8 and lower:

1. JSON generation bugs
  * `'` and `&` (apostrophe and ampersand) characters are escaped by backslash `\`
1. JSON parsing bugs
  * When [setting qBittorent preferences](#setpref) JSON values, containing `:` semicolons will be disregarded; this mostly affects Windows users, whose paths start with `DiskName:\`
  * When [setting qBittorent preferences](#setpref) JSON bool lists (e.g. `"download_in_scan_dirs":[false,true]`) will be treated as all bool values in the list are `false`, this doesn't affect bool values outside JSON lists
