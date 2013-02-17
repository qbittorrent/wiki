**This is a work in progress, it's still not finished**

# Table of Contents #

1. [Authorization](#auth)
1. [GET methods](#GET)
  1. [Shutdown qBittorrent](#shutdown)
  1. [Get torrent list](#torrentlist)
  1. [Get torrent generic properties](#propsgen)
  1. [Get torrent trackers](#propstrack)
  1. [Get torrent contents](#propscont)
  1. [Get global transfer info](#globalinfo)
  1. [Get qBittorrent preferences](#prefget)
1. [POST methods](#POST)
  1. [Download torrent from URL](#dlweb)
  1. Upload torrent from disk
1. Unsorted/Currently unknown/WIP
  1. Add trackers to torrent
  1. Resume all torrents
  1. Pause all torrents
  1. Set prefernces ?setPreferences?
  1. Set file priority
  1. ?getGlobalUpLimit?
  1. ?getGlobalDlLimit?
  1. ?getTorrentUpLimit?
  1. ?getTorrentDlLimit?
  1. ?setTorrentUpLimit?
  1. ?setTorrentDlLimit?
  1. ?setGlobalUpLimit?
  1. ?setGlobalDlLimit?
  1. ?pause?
  1. ?delete?
  1. ?deletePerm?
  1. ?increasePrio?
  1. ?decreasePrio?
  1. ?topPrio?
  1. ?topPrio?
  1. ?recheck?
  1. Get torrent contents

***

# <a id="auth"></a>Authorization #

Authorization requires using `Authorization` header inside GET/POST requests. qBittorrent requires Digest Authorization type with MD5 has generator.

1. Digest Authorization standard

  [This page](http://www.w3.org/People/Raggett/security/draft-ietf-http-digest-aa-00.txt) describes how this auth method should ideally work. The most important thing there is `response` field generation.
  Response is generated like this: `MD5 ( A1 + ':' + nonce + ':' + A2 )` where <br/>
  A1 is `MD5 (username + ':' + realm + ':' + password)` <br/>
  A2 is `MD5 (<method type string (e.g. GET or POST)> + ':' + uri)`

1. Server Auth response

  When you fail Authorization or don't supply required header qBittorrent will send the following reply (example):

  ```
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
      
      ```
HTTP/1.1 401 Unauthorized
WWW-Authenticate: Digest realm="Web UI Access", nonce="a3f396f2dcc1cafae73637e2ac321134", opaque="a3f396f2dcc1cafae73637e2ac321134", stale="false", algorithm="MD5", qop="auth"
      ```

      Client request:

      ```
GET / HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Authorization: Digest username="admin", realm="Web UI Access", nonce="a3f396f2dcc1cafae73637e2ac321134", uri="/", response="4067cfe4c029cd00b56076c78abd034c"
      ```

# <a id="GET"></a> GET methods #

### <a id="shutdown"></a>Shutdown qBittorrent ###

```
GET /command/shutdown HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Authorization: your_auth_string
```

### <a id="torrentlist"></a>Get torrent list ###

```
GET /json/torrents HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Authorization: your_auth_string
```
Server will return the following reply (example):

```
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

### <a id="propsgen"></a>Get torrent generic properties ###

Requires known torrent hash, get 'em from [torrent list](#torrentlist).

```
GET /json/propertiesGeneral/fae6e49afa359ab07c3ef169438ff95d870bc178 HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Authorization: your_auth_string
```

If your torrent hash is invalid server will reply with:

```
HTTP/1.1 200 OK
content-type: text/javascript
content-length: 0
```

Otherwise server will return the following reply (example):

```
HTTP/1.1 200 OK
content-type: text/javascript
content-length: lenth

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

### <a id="propstrack"></a>Get torrent trackers ###

Requires known torrent hash, get 'em from [torrent list](#torrentlist).

```
GET http://127.0.0.1/json/propertiesTrackers/fae6e49afa359ab07c3ef169438ff95d870bc178 HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Authorization: your_auth_string
```

If your torrent hash is invalid server will reply with:

```
HTTP/1.1 200 OK
content-type: text/javascript
content-length: 2

[]
```

Otherwise server will return the following reply (example):

```
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

### <a id="propscont"></a>Get torrent contents ###

Requires known torrent hash, get 'em from [torrent list](#torrentlist).

```
GET http://127.0.0.1/json/propertiesFiles/fae6e49afa359ab07c3ef169438ff95d870bc178 HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Authorization: your_auth_string
```

If your torrent hash is invalid server will reply with:

```
HTTP/1.1 200 OK
content-type: text/javascript
content-length: 0
```

Otherwise server will return the following reply (example):

```
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

### <a id="globalinfo"></a>Get global transfer info ###

This method returns info you usually see in qBt status bar.

```
GET http://127.0.0.1/json/transferInfo HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Authorization: your_auth_string
```

Server reply (example):

```
HTTP/1.1 200 OK
content-type: text/javascript
content-length: length

{"dl_info":"Приём: 0 Б/с - Передано: 0 Б","up_info":"Отдача: 209.9 КиБ/с - Передано: 7.2 ГиБ"}
```
where

`dl_info` - (translated string) contains current global downalod speed and global amount of data downaloded during this session<br/>
`up_info` - (translated string) contains current global upload speed and global amount of data uploaded during this session<br/>

### <a id="prefget"></a>Get qBittorrent preferences ###

```
GET http://127.0.0.1/json/preferences HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Authorization: your_auth_string
```

Server reply; contents may vary depending on which settings are present in qBittorrent.ini (example):

```
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

**WOK IN PROGRESS - TOO MANY VALUES TO DESCRIBE**

# <a id="POST"></a>POST methods #

**Please adjust you auth string aacordingly for POST methods.**

### <a id="dlweb"></a>Download torrent from URL ###

This method can add torrents from urls and magnet links. BC links are also supported.

```
POST http://127.0.0.1/command/download HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Authorization: Digest username="admin", realm="Web UI Access", nonce="a3f396f2dcc1cafae73637e2ac321134", uri="/", response="3f6b2f30c95dfed64c0e835119ce48d0"
Content-Type: application/x-www-form-urlencoded
Content-Length: length

urls=http://www.nyaa.eu/?page=download%26tid=305093
     http://www.nyaa.eu/?page=download%26tid=305255
     magnet:?xt=urn:btih:4c284ebef5bf0d967e2e174cfe825d9fb40ae5e1%26dn=QBittorrent+2.8.4+Win7+Vista+64+working+version%26tr=udp%3A%2F%2Ftracker.openbittorrent.com%3A80%26tr=udp%3A%2F%2Ftracker.publicbt.com%3A80%26tr=udp%3A%2F%2Ftracker.istole.it%3A6969%26tr=udp%3A%2F%2Ftracker.ccc.de%3A80
```

Please note that:

1. `Content-Type: application/x-www-form-urlencoded` is required
1. `urls` contains a list of links; each links goes on a new line
1. `http://`, `https://`, `magnet:` and `bc://bt/` links are supported
1. Links' contents must be escaped, e.g. `&` becomes `%26` (don't know about other characters but ampersand **MUST** be escaped)

No matter if successful or not server will return the following reply:

```
HTTP/1.1 200 OK
```
