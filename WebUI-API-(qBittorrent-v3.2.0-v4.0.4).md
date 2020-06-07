This WebUI API documentation applies to qBittorrent v3.2.0-v4.0.4. For other API versions, visit [WebUI API](https://github.com/qbittorrent/qBittorrent/wiki#WebUI-API).

# Table of Contents #

1. [Changes](#changes)
1. [Authorization](#authorization)
   1. [Login](#login)
   1. [Logout](#logout)
1. [GET methods](#get-methods)
   1. [Get API version](#get-api-version)
   1. [Get minimum API version](#get-minimum-api-version)
   1. [Get qBittorrent version](#get-qbittorrent-version)
   1. [Shutdown qBittorrent](#shutdown-qbittorrent)
   1. [Get torrent list](#get-torrent-list)
   1. [Get torrent generic properties](#get-torrent-generic-properties)
   1. [Get torrent trackers](#get-torrent-trackers)
   1. [Get torrent web seeds](#get-torrent-web-seeds)
   1. [Get torrent contents](#get-torrent-contents)
   1. [Get torrent pieces' states](#get-torrent-pieces-states)
   1. [Get torrent pieces' hashes](#get-torrent-pieces-hashes)
   1. [Get global transfer info](#get-global-transfer-info)
   1. [Get qBittorrent preferences](#get-qbittorrent-preferences)
   1. [Get partial data](#get-partial-data)
   1. [Get log](#get-log)
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
   1. [Set torrent location](#set-torrent-location)
   1. [Set torrent name](#set-torrent-name)
   1. [Set torrent category](#set-torrent-category)
   1. [Add new category](#add-new-category)
   1. [Remove categories](#remove-categories)
   1. [Set automatic torrent management](#set-automatic-torrent-management)
   1. [Set qBittorrent preferences](#set-qbittorrent-preferences)
   1. [Get alternative speed limits state](#get-alternative-speed-limits-state)
   1. [Toggle alternative speed limits](#toggle-alternative-speed-limits)
   1. [Toggle sequential download](#toggle-sequential-download)
   1. [Set first/last piece priority](#set-firstlast-piece-priority)
   1. [Set force start](#set-force-start)
   1. [Set super seeding](#set-super-seeding)
   1. [Pause all the torrents](#pause-all-the-torrents)
   1. [Resume all the torrents](#resume-all-the-torrents)

***

# Changes #

`APIX`, where `X` is a number representing the API version, specifies the first API version in which the marked entry is available.

`APIX-APIY`, where `X` and `Y` are two different API versions, represents a range where `APIX` and `APIY` are respectively the first and last API version in which the marked entry is available.

`API2` is implied and not reported.


* Changes in `API2`:
  * The authentication is now cookie based
  * Sizes are expressed in bytes, speed rates in bytes per seconds, dates as unix timestamps and are all reported as integers with no hardcoded unit
  * `/json/` was replaced with `/query/` in all the requests
  * `/command/getTorrentUpLimit|getTorrentDlLimit|setTorrentUpLimit|setTorrentDlLimit` were replaced by `/command/getTorrentsUpLimit|getTorrentsDlLimit|setTorrentsUpLimit|setTorrentsDlLimit` to handle multiple torrents
  * URIs/parameters are case sensitive
  * The server replies with an error in case of bad requests, unless differently stated.

* Changes between `API3` and `API4`:
  * `application/json` is the MIME type of the replies that had `text/javascript` as MIME type.

* Changes in `API14`:
  * The `Referer` header is now expected in all HTTP requests. qBittorrent will drop any request sent without the referer header.

# Authorization #

qBittorrent uses a cookie based authentication.

### Login ###

```http
POST /login HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Content-Type: application/x-www-form-urlencoded
Content-Length: length

username=admin&password=admin
```

Server reply (example):
```http
Content-Length: 3
Content-Type: text/plain; charset=UTF-8
Set-Cookie: SID=3133exaykXDX0mUQRDukIr9YUi0EchFY; path=/
```
You must supply the cookie whenever you want to perform an operation that requires authentication.

Example showing how to login and execute a command that requires authentication using `curl`:
```sh
$ curl -i --header 'Referer: http://localhost:8080' --data 'username=admin&password=adminadmin' http://localhost:8080/login
HTTP/1.1 200 OK
Content-Encoding:
Content-Length: 3
Content-Type: text/plain; charset=UTF-8
Set-Cookie: SID=hBc7TxF76ERhvIw0jQQ4LZ7Z1jQUV0tQ; path=/
$ curl http://localhost:8080/query/torrents --cookie "SID=hBc7TxF76ERhvIw0jQQ4LZ7Z1jQUV0tQ"
```

Note: Set `Referer` or `Origin` header to the exact same domain and port as used in the HTTP query `Host` header.

### Logout ###

```http
POST /logout HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Cookie: SID=your_sid
Content-Type: application/x-www-form-urlencoded
Content-Length: length
```

# GET methods #

### Get API version ###

Get the current API version

```http
GET /version/api HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
```

Server reply (example):
```http
HTTP/1.1 200 OK
Content-Encoding:
Content-Length: 1
Content-Type: text/plain; charset=UTF-8

2
```

### Get minimum API version ###

Get the minimum API version supported. Any application designed to work with an API version greater than or equal to the minimum API version is guaranteed to work.

```http
GET /version/api_min HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
```

Server reply (example):
```http
HTTP/1.1 200 OK
Content-Encoding:
Content-Length: 1
Content-Type: text/plain; charset=UTF-8

2
```

### Get qBittorrent version ###

```http
GET /version/qbittorrent HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
```

Server reply (example):
```http
HTTP/1.1 200 OK
Content-Encoding:
Content-Length: 1
Content-Type: text/plain; charset=UTF-8

v3.2.0
```

### Shutdown qBittorrent ###

```http
GET /command/shutdown HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Cookie: SID=your_sid
```

Server reply (example):
```http
HTTP/1.1 200 OK
Content-Encoding:
```

### Get torrent list ###

```http
GET /query/torrents HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Cookie: SID=your_sid
```

Params:

Param     | Description
----------|------------
`filter`  | Filter torrent list. Allowed filters: `all`, `downloading`, `completed`, `paused`, `active`, `inactive`
`category`| Get torrents with the given category (empty string means "without category"; no "category" param means "any category")
`sort`    | Sort torrents by given key. All the possible keys are listed here below
`reverse` | Enable reverse sorting. Possible values are `true` and `false` (default)
`limit`   | Limit the number of torrents returned
`offset`  | Set offset (if less than 0, offset from end)
`hashes`  | Filter by hashes. Can contain multiple hashes separated by `\|`

Example:
```http
/query/torrents?filter=downloading&category=sample%20category&sort=ratio
```

Server will return the following reply (example):

```http
HTTP/1.1 200 OK
content-type: application/json
content-length: length

[{"dlspeed":9681262,"eta":87,"f_l_piece_prio":false,"force_start":false,"hash":"8c212779b4abde7c6bc608063a0d008b7e40ce32","category":"","name":"debian-8.1.0-amd64-CD-1.iso","num_complete":-1,"num_incomplete":-1,"num_leechs":2,"num_seeds":54,"priority":1,"progress":0.16108787059783936,"ratio":0,"seq_dl":false,"size":657457152,"state":"downloading","super_seeding":false,"upspeed":0},{another_torrent_info}]
```

Property          | Type    | Description
------------------|---------|------------
`hash`            | string  | Torrent hash
`name`            | string  | Torrent name
`size`            | integer | Total size (bytes) of files selected for download
`progress`        | float   | Torrent progress (percentage/100)
`dlspeed`         | integer | Torrent download speed (bytes/s)
`upspeed`         | integer | Torrent upload speed (bytes/s)
`priority`        | integer | Torrent priority. Returns -1 if queuing is disabled or torrent is in seed mode
`num_seeds`       | integer | Number of seeds connected to
`num_complete`    | integer | Number of seeds in the swarm
`num_leechs`      | integer | Number of leechers connected to
`num_incomplete`  | integer | Number of leechers in the swarm
`ratio`           | float   | Torrent share ratio. Max ratio value: 9999.
`eta`             | integer | Torrent ETA (seconds)
`state`           | string  | Torrent state. See table here below for the possible values
`seq_dl`          | bool    | True if sequential download is enabled
`f_l_piece_prio`  | bool    | True if first last piece are prioritized
`category`           | string  | Category of the torrent
`super_seeding`   | bool    | True if super seeding is enabled
`force_start`     | bool    | True if force start is enabled for this torrent

Possible values of `state`:

Value         | Description
--------------|------------
`error`       | Some error occurred, applies to paused torrents
`pausedUP`    | Torrent is paused and has finished downloading
`pausedDL`    | Torrent is paused and has NOT finished downloading
`queuedUP`    | Queuing is enabled and torrent is queued for upload
`queuedDL`    | Queuing is enabled and torrent is queued for download
`uploading`   | Torrent is being seeded and data is being transferred
`stalledUP`   | Torrent is being seeded, but no connection were made
`checkingUP`  | Torrent has finished downloading and is being checked; this status also applies to preallocation (if enabled) and checking resume data on qBt startup
`checkingDL`  | Same as checkingUP, but torrent has NOT finished downloading
`downloading` | Torrent is being downloaded and data is being transferred
`stalledDL`   | Torrent is being downloaded, but no connection were made
`metaDL`      | Torrent has just started downloading and is fetching metadata

### Get torrent generic properties ###

Requires knowing the torrent hash. You can get it from [torrent list](#get-torrent-list).

```http
GET /query/propertiesGeneral/8c212779b4abde7c6bc608063a0d008b7e40ce32 HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Cookie: SID=your_sid
```

If your torrent hash is invalid server will reply with:

```http
HTTP/1.1 200 OK
content-type: application/json
content-length: 0
```

Otherwise server will return the following reply (example):

```http
HTTP/1.1 200 OK
content-type: application/json
content-length: length

{"addition_date":1438429165,"comment":"\"Debian CD from cdimage.debian.org\"","completion_date":1438429234,"created_by":"","creation_date":1433605214,"dl_limit":-1,"dl_speed":0,"dl_speed_avg":9736015,"eta":8640000,"last_seen":1438430354,"nb_connections":3,"nb_connections_limit":250,"peers":1,"peers_total":89,"piece_size":524288,"pieces_have":1254,"pieces_num":1254,"reannounce":672,"save_path":"/Downloads/debian-8.1.0-amd64-CD-1.iso","seeding_time":1128,"seeds":1,"seeds_total":254,"share_ratio":0.00072121022562178299,"time_elapsed":1197,"total_downloaded":681521119,"total_downloaded_session":681521119,"total_size":657457152,"total_uploaded":491520,"total_uploaded_session":491520,"total_wasted":23481724,"up_limit":-1,"up_speed":0,"up_speed_avg":410}
```

Property                  | Type    | Description
--------------------------|---------|------------
`save_path`               | string  | Torrent save path
`creation_date`           | integer | Torrent creation date (Unix timestamp)
`piece_size`              | integer | Torrent piece size (bytes)
`comment`                 | string  | Torrent comment
`total_wasted`            | integer | Total data wasted for torrent (bytes)
`total_uploaded`          | integer | Total data uploaded for torrent (bytes)
`total_uploaded_session`  | integer | Total data uploaded this session (bytes)
`total_downloaded`        | integer | Total data uploaded for torrent (bytes)
`total_downloaded_session`| integer | Total data downloaded this session (bytes)
`up_limit`                | integer | Torrent upload limit (bytes/s)
`dl_limit`                | integer | Torrent download limit (bytes/s)
`time_elapsed`            | integer | Torrent elapsed time (seconds)
`seeding_time`            | integer | Torrent elapsed time while complete (seconds)
`nb_connections`          | integer | Torrent connection count
`nb_connections_limit`    | integer | Torrent connection count limit
`share_ratio`             | float   | Torrent share ratio
`addition_date` `API4`    | integer | When this torrent was added (unix timestamp)
`completion_date` `API4`  | integer | Torrent completion date (unix timestamp)
`created_by` `API4`       | string  | Torrent creator
`dl_speed_avg` `API4`     | integer | Torrent average download speed (bytes/second)
`dl_speed` `API4`         | integer | Torrent download speed (bytes/second)
`eta` `API4`              | integer | Torrent ETA (seconds)
`last_seen` `API4`        | integer | Last seen complete date (unix timestamp)
`peers` `API4`            | integer | Number of peers connected to
`peers_total` `API4`      | integer | Number of peers in the swarm
`pieces_have` `API4`      | integer | Number of pieces owned
`pieces_num` `API4`       | integer | Number of pieces of the torrent
`reannounce` `API4`       | integer | Number of seconds until the next announce
`seeds` `API4`            | integer | Number of seeds connected to
`seeds_total` `API4`      | integer | Number of seeds in the swarm
`total_size` `API4`       | integer | Torrent total size (bytes)
`up_speed_avg` `API4`     | integer | Torrent average upload speed (bytes/second)
`up_speed` `API4`         | integer | Torrent upload speed (bytes/second)


NB: `-1` is returned when the value is not known if the type is integer.

### Get torrent trackers ###

Requires knowing the torrent hash. You can get it from [torrent list](#get-torrent-list).

```http
GET /query/propertiesTrackers/8c212779b4abde7c6bc608063a0d008b7e40ce32 HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Cookie: SID=your_sid
```

If your torrent hash is invalid server will reply with:

```http
HTTP/1.1 200 OK
content-type: application/json
content-length: 2
```

Otherwise server will return the following reply (example):

```http
HTTP/1.1 200 OK
content-type: application/json
content-length: length

[{"msg":"","num_peers":100,"status":"Working","url":"http://bttracker.debian.org:6969/announce"},{another_tracker_info}]
```

Property      | Type     | Description
--------------|----------|-------------
`url`         | string   | Tracker url
`status`      | string   | Tracker status (translated string). See the table here below for the possible values
`num_peers`   | integer  | Number of peers for current torrent reported by the tracker
`msg`         | string   | Tracker message (there is no way of knowing what this message is - it's up to tracker admins)

Possible values of `status` (translated):

Value               | Description
--------------------|------------
`Working`           | Tracker has been contacted and is working
`Updating...`       | Tracker is currently being updated
`Not working`       | Tracker has been contacted, but it is not working (or doesn't send proper replies)
`Not contacted yet` | Tracker has not been contacted yet

### Get torrent web seeds ###

`API3`

Requires knowing the torrent hash. You can get it from [torrent list](#get-torrent-list).

```http
GET /query/propertiesWebSeeds/8c212779b4abde7c6bc608063a0d008b7e40ce32 HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Cookie: SID=your_sid
```

```http
HTTP/1.1 200 OK
content-type: application/json
content-length: length

[{"url":"http://some_url/"},{"url":"http://some_other_url/"}]
```

Property      | Type     | Description
--------------|----------|------------
`url`         | string   | URL of the web seed

### Get torrent contents ###

Requires knowing the torrent hash. You can get it from [torrent list](#get-torrent-list).

```http
GET /query/propertiesFiles/8c212779b4abde7c6bc608063a0d008b7e40ce32 HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Cookie: SID=your_sid
```

If your torrent hash is invalid server will reply with:

```http
HTTP/1.1 200 OK
content-type: application/json
content-length: 0
```

Otherwise server will return the following reply (example):

```http
HTTP/1.1 200 OK
content-type: application/json
content-length: length

[{"is_seed":false,"name":"debian-8.1.0-amd64-CD-1.iso","piece_range":[0,1253],"priority":4,"progress":0,"size":657457152}]
```

Property       | Type          | Description
---------------|---------------|-------------
`name`         | string        | File name (including relative path)
`size`         | integer       | File size (bytes)
`progress`     | float         | File progress (percentage/100)
`priority`     | integer       | File priority. See possible values here below
`is_seed`      | bool          | True if file is seeding/complete
`piece_range`  | integer array | The first number is the starting piece index and the second number is the ending piece index (inclusive)

Possible values of `priority`:

Value      | Description
-----------|------------
`0`        | Do not download
`1`        | Normal priority
`6`        | High priority
`7`        | Maximal priority

### Get torrent pieces' states ###

Returns an array of states (integers) of all pieces (in order) of a specific torrent.

Requires knowing the torrent hash. You can get it from [torrent list](#get-torrent-list).

```http
GET /query/getPieceStates/8c212779b4abde7c6bc608063a0d008b7e40ce32 HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Cookie: SID=your_sid
```

If your torrent hash is invalid server will reply with:

```http
HTTP/1.1 200 OK
content-type: application/json
content-length: 0
```

Otherwise server will return the following reply (example):

```http
HTTP/1.1 200 OK
content-type: application/json
content-length: length

[0,0,2,1,0,0,2,1]
```

Value meanings are defined as below:

Value      | Description
-----------|------------
`0`        | Not downloaded yet
`1`        | Now downloading
`2`        | Already downloaded

### Get torrent pieces' hashes ###

Returns an array of hashes (strings) of all pieces (in order) of a specific torrent.

Requires knowing the torrent hash. You can get it from [torrent list](#get-torrent-list).

```http
GET /query/getPieceHashes/8c212779b4abde7c6bc608063a0d008b7e40ce32 HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Cookie: SID=your_sid
```

If your torrent hash is invalid server will reply with:

```http
HTTP/1.1 200 OK
content-type: application/json
content-length: 0
```

Otherwise server will return the following reply (example):

```http
HTTP/1.1 200 OK
content-type: application/json
content-length: length

["54eddd830a5b58480a6143d616a97e3a6c23c439","f8a99d225aa4241db100f88407fc3bdaead583ab","928fb615b9bd4dd8f9e9022552c8f8f37ef76f58"]
```

### Get global transfer info ###

This method returns info you usually see in qBt status bar.

```http
GET /query/transferInfo HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Cookie: SID=your_sid
```

Server reply (example):

```http
HTTP/1.1 200 OK
content-type: application/json
content-length: length

{"connection_status":"connected","dht_nodes":386,"dl_info_data":681521119,"dl_info_speed":0,"dl_rate_limit":0,"up_info_data":10747904,"up_info_speed":0,"up_rate_limit":1048576}
```

Property            | Type    | Description
--------------------|---------|------------
`dl_info_speed`     | integer | Global download rate (bytes/s)
`dl_info_data`      | integer | Data downloaded this session (bytes)
`up_info_speed`     | integer | Global upload rate (bytes/s)
`up_info_data`      | integer | Data uploaded this session (bytes)
`dl_rate_limit`     | integer | Download rate limit (bytes/s)
`up_rate_limit`     | integer | Upload rate limit (bytes/s)
`dht_nodes`         | integer | DHT nodes connected to
`connection_status` | string  | Connection status. See possible values here below

In addition to the above in partial data requests (see [Get partial data](#get-partial-data) for more info):

Property               | Type    | Description
-----------------------|---------|------------
`queueing`             | bool    | True if torrent queueing is enabled
`use_alt_speed_limits` | bool    | True if alternative speed limits are enabled
`refresh_interval`     | integer | Transfer list refresh interval (milliseconds)

Possible values of `connection_status`:

Value               |
--------------------|
`connected`         |
`firewalled`        |
`disconnected`      |

### Get qBittorrent preferences ###

```http
GET /query/preferences HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Cookie: SID=your_sid
```

Server reply; contents may vary depending on which settings are present in qBittorrent.ini (example):

```http
HTTP/1.1 200 OK
content-type: application/json
content-length: length

{"locale":"ru_RU","save_path":"C:/Users/Dayman/Downloads","temp_path_enabled":false,"temp_path":"C:/Users/Dayman/Documents/Downloads/temp","scan_dirs":["D:/Browser Downloads"],"download_in_scan_dirs":[false],"export_dir_enabled":false,"export_dir":"","mail_notification_enabled":false,"mail_notification_email":"","mail_notification_smtp":"smtp.changeme.com","mail_notification_ssl_enabled":false,"mail_notification_auth_enabled":false,"mail_notification_username":"","mail_notification_password":"","autorun_enabled":false,"autorun_program":"","preallocate_all":false,"queueing_enabled":true,"max_active_downloads":2,"max_active_torrents":200,"max_active_uploads":200,"dont_count_slow_torrents":false,"incomplete_files_ext":false,"listen_port":31498,"upnp":false,"dl_limit":3072,"up_limit":3072,"max_connec":500,"max_connec_per_torrent":100,"max_uploads_per_torrent":15,"enable_utp":true,"limit_utp_rate":false,"limit_tcp_overhead":true,"alt_dl_limit":1024,"alt_up_limit":2048,"scheduler_enabled":false,"schedule_from_hour":8,"schedule_from_min":0,"schedule_to_hour":20,"schedule_to_min":0,"scheduler_days":0,"dht":true,"dhtSameAsBT":true,"dht_port":6881,"pex":true,"lsd":true,"encryption":0,"anonymous_mode":false,"proxy_type":-1,"proxy_ip":"0.0.0.0","proxy_port":8080,"proxy_peer_connections":false,"proxy_auth_enabled":false,"proxy_username":"","proxy_password":"","ip_filter_enabled":false,"ip_filter_path":null,"web_ui_port":80,"web_ui_username":"admin","web_ui_password":"8888efb275743684292cff99f57867a9","bypass_local_auth":false,"use_https":false,"ssl_key":"","ssl_cert":"","dyndns_enabled":false,"dyndns_service":0,"dyndns_username":"","dyndns_password":"","dyndns_domain":"changeme.dyndns.org"}
```
where

Property                          | Type    | Description
----------------------------------|---------|------------
`locale`                          | string  | Currently selected language (e.g. en_GB for English)
`save_path`                       | string  | Default save path for torrents, separated by slashes
`temp_path_enabled`               | bool    | True if folder for incomplete torrents is enabled
`temp_path`                       | string  | Path for incomplete torrents, separated by slashes
`scan_dirs`                       | string  | List of watch folders to add torrent automatically; slashes are used as path separators; list entries are separated by commas
`download_in_scan_dirs`           | bool    | True if torrents should be downloaded to watch folder; list entries are separated by commas
`export_dir_enabled`              | bool    | True if .torrent file should be copied to export directory upon adding
`export_dir`                      | string  | Path to directory to copy .torrent files if `export_dir_enabled` is enabled; path is separated by slashes
`mail_notification_enabled`       | bool    | True if e-mail notification should be enabled
`mail_notification_email`         | string  | e-mail to send notifications to
`mail_notification_smtp`          | string  | smtp server for e-mail notifications
`mail_notification_ssl_enabled`   | bool    | True if smtp server requires SSL connection
`mail_notification_auth_enabled`  | bool    | True if smtp server requires authentication
`mail_notification_username`      | string  | Username for smtp authentication
`mail_notification_password`      | string  | Password for smtp authentication
`autorun_enabled`                 | bool    | True if external program should be run after torrent has finished downloading
`autorun_program`                 | string  | Program path/name/arguments to run if `autorun_enabled` is enabled; path is separated by slashes; you can use `%f` and `%n` arguments, which will be expanded by qBittorent as path_to_torrent_file and torrent_name (from the GUI; not the .torrent file name) respectively
`preallocate_all`                 | bool    | True if file preallocation should take place, otherwise sparse files are used
`queueing_enabled`                | bool    | True if torrent queuing is enabled
`max_active_downloads`            | integer | Maximum number of active simultaneous downloads
`max_active_torrents`             | integer | Maximum number of active simultaneous downloads and uploads
`max_active_uploads`              | integer | Maximum number of active simultaneous uploads
`dont_count_slow_torrents`        | bool    | If true torrents w/o any activity (stalled ones) will not be counted towards `max_active_*` limits; see [dont_count_slow_torrents](https://www.libtorrent.org/reference-Settings.html#dont_count_slow_torrents) for more information
`max_ratio_enabled` `API3`        | bool    | True if share ratio limit is enabled
`max_ratio` `API3`                | float   | Get the global share ratio limit
`max_ratio_act` `API3`            | bool    | Action performed when a torrent reaches the maximum share ratio. See list of possible values here below.
`incomplete_files_ext`            | bool    | If true `.!qB` extension will be appended to incomplete files
`listen_port`                     | integer | Port for incoming connections
`upnp`                            | bool    | True if UPnP/NAT-PMP is enabled
`random_port` `API3`              | bool    | True if the port is randomly selected
`dl_limit`                        | integer | Global download speed limit in KiB/s; `-1` means no limit is applied
`up_limit`                        | integer | Global upload speed limit in KiB/s; `-1` means no limit is applied
`max_connec`                      | integer | Maximum global number of simultaneous connections
`max_connec_per_torrent`          | integer | Maximum number of simultaneous connections per torrent
`max_uploads` `API3`              | integer | Maximum number of upload slots
`max_uploads_per_torrent`         | integer | Maximum number of upload slots per torrent
`enable_utp`                      | bool    | True if uTP protocol should be enabled; this option is only available in qBittorent built against libtorrent version 0.16.X and higher
`limit_utp_rate`                  | bool    | True if `[du]l_limit` should be applied to uTP connections; this option is only available in qBittorent built against libtorrent version 0.16.X and higher
`limit_tcp_overhead`              | bool    | True if `[du]l_limit` should be applied to estimated TCP overhead (service data: e.g. packet headers)
`alt_dl_limit`                    | integer | Alternative global download speed limit in KiB/s
`alt_up_limit`                    | integer | Alternative global upload speed limit in KiB/s
`scheduler_enabled`               | bool    | True if alternative limits should be applied according to schedule
`schedule_from_hour`              | integer | Scheduler starting hour
`schedule_from_min`               | integer | Scheduler starting minute
`schedule_to_hour`                | integer | Scheduler ending hour
`schedule_to_min`                 | integer | Scheduler ending minute
`scheduler_days`                  | integer | Scheduler days. See possible values here below
`dht`                             | bool    | True if DHT is enabled
`dhtSameAsBT`                     | bool    | True if DHT port should match TCP port
`dht_port`                        | integer | DHT port if `dhtSameAsBT` is false
`pex`                             | bool    | True if PeX is enabled
`lsd`                             | bool    | True if LSD is enabled
`encryption`                      | integer | See list of possible values here below
`anonymous_mode`                  | bool    | If true anonymous mode will be enabled; read more [here](Anonymous-Mode); this option is only available in qBittorent built against libtorrent version 0.16.X and higher
`proxy_type`                      | integer | See list of possible values here below
`proxy_ip`                        | string  | Proxy IP address or domain name
`proxy_port`                      | integer | Proxy port
`proxy_peer_connections`          | bool    | True if peer and web seed connections should be proxified; this option will have any effect only in qBittorent built against libtorrent version 0.16.X and higher
`force_proxy` `API3`              | bool    | True if the connections not supported by the proxy are disabled
`proxy_auth_enabled`              | bool    | True proxy requires authentication; doesn't apply to SOCKS4 proxies
`proxy_username`                  | string  | Username for proxy authentication
`proxy_password`                  | string  | Password for proxy authentication
`ip_filter_enabled`               | bool    | True if external IP filter should be enabled
`ip_filter_path`                  | string  | Path to IP filter file (.dat, .p2p, .p2b files are supported); path is separated by slashes
`ip_filter_trackers` `API3`       | bool    | True if IP filters are applied to trackers
`web_ui_port`                     | integer | WebUI port
`web_ui_upnp` `API4`              | bool    | True if UPnP is used for the WebUI port
`web_ui_username`                 | string  | WebUI username
`web_ui_password`                 | string  | MD5 hash of WebUI password; hash is generated from the following string: `username:Web UI Access:plain_text_web_ui_password`
`web_ui_csrf_protection_enabled`  | bool    | True if WebUI CSRF protection is enabled
`web_ui_clickjacking_protection_enabled` | bool | True if WebUI clickjacking protection is enabled
`bypass_local_auth`               | bool    | True if authentication challenge for loopback address (127.0.0.1) should be disabled
`bypass_auth_subnet_whitelist_enabled` | bool | True if webui authentication should be bypassed for clients whose ip resides within (at least) one of the subnets on the whitelist
`bypass_auth_subnet_whitelist` | string | (White)list of ipv4/ipv6 subnets for which webui authentication should be bypassed; list entries are separated by commas
`use_https`                       | bool    | True if WebUI HTTPS access is enabled
`ssl_key`                         | string  | SSL keyfile contents (this is a not a path)
`ssl_cert`                        | string  | SSL certificate contents (this is a not a path)
`dyndns_enabled`                  | bool    | True if server DNS should be updated dynamically
`dyndns_service`                  | integer | See list of possible values here below
`dyndns_username`                 | string  | Username for DDNS service
`dyndns_password`                 | string  | Password for DDNS service
`dyndns_domain`                   | string  | Your DDNS domain name

Possible values of `scheduler_days`:

Value  | Description
-------|------------
`0`    | Every day
`1`    | Every weekday
`2`    | Every weekend
`3`    | Every Monday
`4`    | Every Tuesday
`5`    | Every Wednesday
`6`    | Every Thursday
`7`    | Every Friday
`8`    | Every Saturday
`9`    | Every Sunday

Possible values of `encryption`:

Value  | Description
-------|------------
`0`    | Prefer encryption
`1`    | Force encryption on
`2`    | Force encryption off

NB: the first options allows you to use both encrypted and unencrypted connections (this is the default); other options are mutually exclusive: e.g. by forcing encryption on you won't be able to use unencrypted connections and vice versa.

Possible values of `proxy_type`:

Value | Description
------|------------
`-1`  | Proxy is disabled
`1`   | HTTP proxy without authentication
`2`   | SOCKS5 proxy without authentication
`3`   | HTTP proxy with authentication
`4`   | SOCKS5 proxy with authentication
`5`   | SOCKS4 proxy without authentication

Possible values of `dyndns_service`:

Value  | Description
-------|------------
`0`    | Use DyDNS
`1`    | Use NOIP

Possible values of `max_ratio_act`:

Value | Description
------|------------
`0`   | Pause torrent
`1`   | Remove torrent

### Get partial data ###

Request only for changes since the last request.

```http
GET /sync/maindata HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Cookie: SID=your_sid
```

Params:

Param | Description
------|------------
`rid` | Response ID. If not provided, `rid=0` will be assumed. If the given `rid` is different from the one of last server reply, `full_update` will be `true` (see the server reply details for more info)

Example:
```http
http://127.0.0.1/sync/maindata?rid=14
```

Server reply (example):
```http
HTTP/1.1 200 OK
content-type: application/json
content-length: length

{"rid":15,"torrents":{"8c212779b4abde7c6bc608063a0d008b7e40ce32":{"state":"pausedUP"}}}
```

Property                      | Type    | Description
------------------------------|---------|------------
`rid`                         | integer | Response ID
`full_update`                 | bool    | Whether the response contains all the data or partial data
`torrents`                    | object  | Property: torrent hash, value: same as [torrent list](#get-torrent-list)
`torrents_removed`            | array   | List of hashes of torrents removed since last request
`categories`                  | array   | List of categories added since last request
`categories_removed`          | array   | List of categories removed since last request
`queueing`                    | bool    | Priority system usage flag
`server_state`                | object  | Same as [global transfer info](#get-global-transfer-info)

### Get log ###

```http
GET /query/getLog HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Cookie: SID=your_sid
```

Params:

Param           | Type    | Description
----------------|---------|------------
`normal`        | bool    | Include normal messages (default: `true`)
`info`          | bool    | Include info messages (default: `true`)
`warning`       | bool    | Include warning messages (default: `true`)
`critical`      | bool    | Include critical messages (default: `true`)
`last_known_id` | integer | Exclude messages with "message id" <= `last_known_id` (default: `-1`)

Example:
```http
/query/getLog?normal=true&info=true&warning=true&critical=true&last_known_id=-1
```

Server will return the following reply (example):

```http
HTTP/1.1 200 OK
content-type: application/json
content-length: length

[{"id":0,"message":"qBittorrent v3.4.0 started","timestamp":1507969127860,"type":1},{"id":1,"message":"qBittorrent is trying to listen on any interface port: 19036","timestamp":1507969127869,"type":2},{"id":2,"message":"Peer ID: -qB3400-","timestamp":1507969127870,"type":1},{"id":3,"message":"HTTP User-Agent is 'qBittorrent/3.4.0'","timestamp":1507969127870,"type":1},{"id":4,"message":"DHT support [ON]","timestamp":1507969127871,"type":2},{"id":5,"message":"Local Peer Discovery support [ON]","timestamp":1507969127871,"type":2},{"id":6,"message":"PeX support [ON]","timestamp":1507969127871,"type":2},{"id":7,"message":"Anonymous mode [OFF]","timestamp":1507969127871,"type":2},{"id":8,"message":"Encryption support [ON]","timestamp":1507969127871,"type":2},{"id":9,"message":"Embedded Tracker [OFF]","timestamp":1507969127871,"type":2},{"id":10,"message":"UPnP / NAT-PMP support [ON]","timestamp":1507969127873,"type":2},{"id":11,"message":"Web UI: Now listening on port 8080","timestamp":1507969127883,"type":1},{"id":12,"message":"Options were saved successfully.","timestamp":1507969128055,"type":1},{"id":13,"message":"qBittorrent is successfully listening on interface :: port: TCP/19036","timestamp":1507969128270,"type":2},{"id":14,"message":"qBittorrent is successfully listening on interface 0.0.0.0 port: TCP/19036","timestamp":1507969128271,"type":2},{"id":15,"message":"qBittorrent is successfully listening on interface 0.0.0.0 port: UDP/19036","timestamp":1507969128272,"type":2}]
```

Property    | Type    | Description
------------|---------|------------
`id`        | integer | ID of the message
`message`   | string  | Text of the message
`timestamp` | integer | Milliseconds since epoch
`type`      | integer | Type of the message: Log::NORMAL: `1`, Log::INFO: `2`, Log::WARNING: `4`, Log::CRITICAL: `8`

# POST methods #

### Download torrent from URL ###

This method can add torrents from URLs. `http://`, `https://`, `magnet:` and `bc://bt/` links are supported.

**After qBittorrent v3.3.1 `API7` (included):**
```http
POST /command/download HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Cookie: SID=your_sid
Content-Type: multipart/form-data; boundary=---------------------------6688794727912
Content-Length: length

-----------------------------6688794727912
Content-Disposition: form-data; name="urls"

https://torcache.net/torrent/3B1A1469C180F447B77021074DBBCCAEF62611E7.torrent
https://torcache.net/torrent/3B1A1469C180F447B77021074DBBCCAEF62611E8.torrent
-----------------------------6688794727912
Content-Disposition: form-data; name="savepath"

C:/Users/qBit/Downloads
-----------------------------6688794727912
Content-Disposition: form-data; name="cookie"

ui=28979218048197
-----------------------------6688794727912
Content-Disposition: form-data; name="category"

movies
-----------------------------6688794727912
Content-Disposition: form-data; name="skip_checking"

true
-----------------------------6688794727912
Content-Disposition: form-data; name="paused"

true
-----------------------------6688794727912
Content-Disposition: form-data; name="root_folder"

true
-----------------------------6688794727912--
```

Property                      | Type    | Description
------------------------------|---------|------------
`urls`                        | string  | URLs separated with newlines
`savepath`                    | string  | (optional) Download folder
`cookie`                      | string  | (optional) Cookie sent to download the .torrent file
`category`                    | string  | (optional) Category for the torrent
`skip_checking`  `API13`      | string  | (optional) Skip hash checking. Possible values are `true`, `false` (default)
`paused`  `API13`             | string  | (optional) Add torrents in the paused state. Possible values are `true`, `false` (default)
`root_folder`  `API15`        | string  | (optional) Create the root folder. Possible values are `true`, `false`, unset (default)
`rename`                      | string  | (optional) Rename torrent
`upLimit`                     | integer | (optional) Set torrent upload speed limit. Unit in bytes/second
`dlLimit`                     | integer | (optional) Set torrent download speed limit. Unit in bytes/second
`sequentialDownload`          | string  | (optional) Enable sequential download. Possible values are `true`, `false` (default)
`firstLastPiecePrio`          | string  | (optional) Prioritize download first last piece. Possible values are `true`, `false` (default)

**Before qBittorrent v3.3.1 `API6`:**
```http
POST /command/download HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Cookie: SID=your_sid
Content-Type: application/x-www-form-urlencoded
Content-Length: length

urls=http://www.nyaa.eu/?page=download%26tid=305093%0Ahttp://www.nyaa.eu/?page=download%26tid=305255%0Amagnet:?xt=urn:btih:4c284ebef5bf0d967e2e174cfe825d9fb40ae5e1%26dn=QBittorrent+2.8.4+Win7+Vista+64+working+version%26tr=udp%3A%2F%2Ftracker.openbittorrent.com%3A80%26tr=udp%3A%2F%2Ftracker.publicbt.com%3A80%26tr=udp%3A%2F%2Ftracker.istole.it%3A6969%26tr=udp%3A%2F%2Ftracker.ccc.de%3A80
```
Please note that:

1. `Content-Type: application/x-www-form-urlencoded` is required
1. `urls` contains a list of links; links are separated with `%0A` (LF newline)
1. Links' contents must be escaped, e.g. `&` becomes `%26` (don't know about other characters but ampersand **MUST** be escaped)

No matter if successful or not server will return the following reply:

```http
HTTP/1.1 200 OK
```

### Upload torrent from disk ###

```http
POST /command/upload HTTP/1.1
Content-Type: multipart/form-data; boundary=-------------------------acebdf13572468
User-Agent: Fiddler
Host: 127.0.0.1
Cookie: SID=your_sid
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

Property                      | Type    | Description
------------------------------|---------|------------
`torrents`                    | raw     | Raw data of torrent file. `torrents` can be presented multiple times.
`savepath`                    | string  | (optional) Download folder
`cookie`                      | string  | (optional) Cookie sent to download the .torrent file
`category`                    | string  | (optional) Category for the torrent
`skip_checking`  `API13`      | string  | (optional) Skip hash checking. Possible values are `true`, `false` (default)
`paused`  `API13`             | string  | (optional) Add torrents in the paused state. Possible values are `true`, `false` (default)
`root_folder`  `API15`        | string  | (optional) Create the root folder. Possible values are `true`, `false`, unset (default)
`rename`                      | string  | (optional) Rename torrent
`upLimit`                     | integer | (optional) Set torrent upload speed limit. Unit in bytes/second
`dlLimit`                     | integer | (optional) Set torrent download speed limit. Unit in bytes/second
`sequentialDownload`          | string  | (optional) Enable sequential download. Possible values are `true`, `false` (default)
`firstLastPiecePrio`          | string  | (optional) Prioritize download first last piece. Possible values are `true`, `false` (default)

### Add trackers to torrent ###

Requires knowing the torrent hash. You can get it from [torrent list](#get-torrent-list).

```http
POST /command/addTrackers HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Cookie: SID=your_sid
Content-Type: application/x-www-form-urlencoded
Content-Length: length

hash=8c212779b4abde7c6bc608063a0d008b7e40ce32&urls=http://192.168.0.1/announce%0Audp://192.168.0.1:3333/dummyAnnounce
```

This adds two trackers to torrent with hash `8c212779b4abde7c6bc608063a0d008b7e40ce32`. Note `%0A` (aka LF newline) between trackers. Ampersand in tracker urls **MUST** be escaped.

No matter if successful or not server will return the following reply:

```http
HTTP/1.1 200 OK
```

### Pause torrent ###

Requires knowing the torrent hash. You can get it from [torrent list](#get-torrent-list).

```http
POST /command/pause HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Cookie: SID=your_sid
Content-Type: application/x-www-form-urlencoded
Content-Length: length

hash=8c212779b4abde7c6bc608063a0d008b7e40ce32
```

No matter if successful or not server will return the following reply:

```http
HTTP/1.1 200 OK
```

### Pause all torrents ###

```http
POST /command/pauseAll HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Cookie: SID=your_sid
Content-Length: 0
```

No matter if successful or not server will return the following reply:

```http
HTTP/1.1 200 OK
```

### Resume torrent ###

Requires knowing the torrent hash. You can get it from [torrent list](#get-torrent-list).

```http
POST /command/resume HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Cookie: SID=your_sid
Content-Type: application/x-www-form-urlencoded
Content-Length: length

hash=8c212779b4abde7c6bc608063a0d008b7e40ce32
```

No matter if successful or not server will return the following reply:

```http
HTTP/1.1 200 OK
```

### Resume all torrents ###

```http
POST /command/resumeAll HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Cookie: SID=your_sid
Content-Length: 0
```

No matter if successful or not server will return the following reply:

```http
HTTP/1.1 200 OK
```

### Delete torrent ###

Requires knowing the torrent hash. You can get it from [torrent list](#get-torrent-list).

```http
POST /command/delete HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Cookie: SID=your_sid
Content-Type: application/x-www-form-urlencoded
Content-Length: length

hashes=8c212779b4abde7c6bc608063a0d008b7e40ce32
```

`hashes` can contain multiple hashes separated by `|`

No matter if successful or not server will return the following reply:

```http
HTTP/1.1 200 OK
```

### Delete torrent with downloaded data ###

Requires knowing the torrent hash. You can get it from [torrent list](#get-torrent-list).

```http
POST /command/deletePerm HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Cookie: SID=your_sid
Content-Type: application/x-www-form-urlencoded
Content-Length: length

hashes=8c212779b4abde7c6bc608063a0d008b7e40ce32
```

`hashes` can contain multiple hashes separated by `|`

No matter if successful or not server will return the following reply:

```http
HTTP/1.1 200 OK
```

### Recheck torrent ###

Requires knowing the torrent hash. You can get it from [torrent list](#get-torrent-list).

```http
POST /command/recheck HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Cookie: SID=your_sid
Content-Type: application/x-www-form-urlencoded
Content-Length: length

hash=8c212779b4abde7c6bc608063a0d008b7e40ce32
```

No matter if successful or not server will return the following reply:

```http
HTTP/1.1 200 OK
```
### Increase torrent priority ###

Requires knowing the torrent hash. You can get it from [torrent list](#get-torrent-list).

```http
POST /command/increasePrio HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Cookie: SID=your_sid
Content-Type: application/x-www-form-urlencoded
Content-Length: length

hashes=8c212779b4abde7c6bc608063a0d008b7e40ce32
```

`hashes` can contain multiple hashes separated by `|`

No matter if successful or not server will return the following reply:

```http
HTTP/1.1 200 OK
```

### Decrease torrent priority ###

Requires knowing the torrent hash. You can get it from [torrent list](#get-torrent-list).

```http
POST /command/decreasePrio HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Cookie: SID=your_sid
Content-Type: application/x-www-form-urlencoded
Content-Length: length

hashes=8c212779b4abde7c6bc608063a0d008b7e40ce32
```

`hashes` can contain multiple hashes separated by `|`

No matter if successful or not server will return the following reply:

```http
HTTP/1.1 200 OK
```

### Maximal torrent priority ###

Requires knowing the torrent hash. You can get it from [torrent list](#get-torrent-list).

```http
POST /command/topPrio HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Cookie: SID=your_sid
Content-Type: application/x-www-form-urlencoded
Content-Length: length

hashes=8c212779b4abde7c6bc608063a0d008b7e40ce32
```

`hashes` can contain multiple hashes separated by `|`

No matter if successful or not server will return the following reply:

```http
HTTP/1.1 200 OK
```

### Minimal torrent priority ###

Requires knowing the torrent hash. You can get it from [torrent list](#get-torrent-list).

```http
POST /command/bottomPrio HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Cookie: SID=your_sid
Content-Type: application/x-www-form-urlencoded
Content-Length: length

hashes=8c212779b4abde7c6bc608063a0d008b7e40ce32
```

`hashes` can contain multiple hashes separated by `|`

No matter if successful or not server will return the following reply:

```http
HTTP/1.1 200 OK
```

### Set file priority ###

Requires knowing the torrent hash. You can get it from [torrent list](#get-torrent-list).

```http
POST /command/setFilePrio HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Cookie: SID=your_sid
Content-Type: application/x-www-form-urlencoded
Content-Length: length

hash=8c212779b4abde7c6bc608063a0d008b7e40ce32&id=0&priority=7
```

Please consult [torrent contents API](#get-torrent-contents) for possible `priority` values. `id` values coresspond to contents returned by [torrent contents API](#get-torrent-contents), e.g. `id=0` for first file, `id=1` for second file, etc.

No matter if successful or not server will return the following reply:

```http
HTTP/1.1 200 OK
```

### Get global download limit ###

```http
POST /command/getGlobalDlLimit HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Cookie: SID=your_sid
Content-Length: 0
```

Server reply (example):

```http
HTTP/1.1 200 OK
content-type: text/plain
content-length: length

3145728
```

`3145728` is the value of current global download speed limit in bytes; this value will be zero if no limit is applied.

### Set global download limit ###

```http
POST /command/setGlobalDlLimit HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Cookie: SID=your_sid
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
POST /command/getGlobalUpLimit HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Cookie: SID=your_sid
Content-Length: 0
```

Server reply (example):

```http
HTTP/1.1 200 OK
content-type: text/plain
content-length: length

3145728
```

`3145728` is the value of current global upload speed limit in bytes; this value will be zero if no limit is applied.

### Set global upload limit ###

```http
POST /command/setGlobalUpLimit HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Cookie: SID=your_sid
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

Requires knowing the torrent hash. You can get it from [torrent list](#get-torrent-list).

```http
POST /command/getTorrentsDlLimit HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Cookie: SID=your_sid
Content-Type: application/x-www-form-urlencoded
Content-Length: length

hashes=8c212779b4abde7c6bc608063a0d008b7e40ce32|284b83c9c7935002391129fd97f43db5d7cc2ba0
```

`hashes` can contain multiple hashes separated by `|`

Server reply (example):

```http
HTTP/1.1 200 OK
content-type: application/json
content-length: length

{"8c212779b4abde7c6bc608063a0d008b7e40ce32":338944,"284b83c9c7935002391129fd97f43db5d7cc2ba0":123}
```

`8c212779b4abde7c6bc608063a0d008b7e40ce32` is the hash of the torrent and `338944` its download speed limit in bytes per second; this value will be zero if no limit is applied.

### Set torrent download limit ###

Requires knowing the torrent hash. You can get it from [torrent list](#get-torrent-list).

```http
POST /command/setTorrentsDlLimit HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Cookie: SID=your_sid
Content-Type: application/x-www-form-urlencoded
Content-Length: length

hashes=8c212779b4abde7c6bc608063a0d008b7e40ce32|284b83c9c7935002391129fd97f43db5d7cc2ba0&limit=131072
```

`hashes` can contain multiple hashes separated by `|`<br />
`limit` is the download speed limit in bytes per second you want to set.

No matter if successful or not server will return the following reply:

```http
HTTP/1.1 200 OK
```

### Get torrent upload limit ###

Requires knowing the torrent hash. You can get it from [torrent list](#get-torrent-list).

```http
POST /command/getTorrentsUpLimit HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Cookie: SID=your_sid
Content-Type: application/x-www-form-urlencoded
Content-Length: length

hashes=8c212779b4abde7c6bc608063a0d008b7e40ce32|284b83c9c7935002391129fd97f43db5d7cc2ba0
```

`hashes` can contain multiple hashes separated by `|`

Server reply (example):

```http
HTTP/1.1 200 OK
content-type: application/json
content-length: length

{"8c212779b4abde7c6bc608063a0d008b7e40ce32":338944,"284b83c9c7935002391129fd97f43db5d7cc2ba0":123}
```

`8c212779b4abde7c6bc608063a0d008b7e40ce32` is the hash of the torrent in the request and `338944` its upload speed limit in bytes per second; this value will be zero if no limit is applied.

### Set torrent upload limit ###

Requires knowing the torrent hash. You can get it from [torrent list](#get-torrent-list).

```http
POST /command/setTorrentsUpLimit HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Cookie: SID=your_sid
Content-Type: application/x-www-form-urlencoded
Content-Length: length

hashes=8c212779b4abde7c6bc608063a0d008b7e40ce32|284b83c9c7935002391129fd97f43db5d7cc2ba0&limit=131072
```

`hashes` can contain multiple hashes separated by `|`<br />
`limit` is the upload speed limit in bytes per second you want to set.

No matter if successful or not server will return the following reply:

```http
HTTP/1.1 200 OK
```

### Set torrent location ###

Requires knowing the torrent hash. You can get it from [torrent list](#get-torrent-list).

```http
POST /command/setLocation HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Cookie: SID=your_sid
Content-Type: application/x-www-form-urlencoded
Content-Length: length

hashes=8c212779b4abde7c6bc608063a0d008b7e40ce32|284b83c9c7935002391129fd97f43db5d7cc2ba0&location=/mnt/nfs/media
```

`hashes` can contain multiple hashes separated by `|`<br />
`location` is the location to download the torrent to. If the location doesn't exist, the torrent's location is unchanged.

No matter if successful or not server will return the following reply:

```http
HTTP/1.1 200 OK
```

### Set torrent name ###

Requires knowing the torrent hash. You can get it from [torrent list](#get-torrent-list).

```http
POST /command/rename HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Cookie: SID=your_sid
Content-Type: application/x-www-form-urlencoded
Content-Length: length

hash=8c212779b4abde7c6bc608063a0d008b7e40ce32&name=This%20is%20a%20test
```

If your torrent hash is invalid or the name is empty, the server will reply with:

```http
HTTP/1.1 400 Incorrect torrent hash or name
```

Otherwise, the server will reply with:

```http
HTTP/1.1 200 OK
```

### Set torrent category ###

Requires knowing the torrent hash. You can get it from [torrent list](#get-torrent-list).

```http
POST /command/setCategory HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Cookie: SID=your_sid
Content-Type: application/x-www-form-urlencoded
Content-Length: length

hashes=8c212779b4abde7c6bc608063a0d008b7e40ce32|284b83c9c7935002391129fd97f43db5d7cc2ba0&category=CategoryName
```

`hashes` can contain multiple hashes separated by `|`<br />
`category` is the torrent category you want to set. If the category doesn't exist, a new category is created.

No matter if successful or not server will return the following reply:

```http
HTTP/1.1 200 OK
```

### Add new category ###

```http
POST /command/addCategory HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Cookie: SID=your_sid
Content-Type: application/x-www-form-urlencoded
Content-Length: length

category=CategoryName
```

`category` is the category you want to create.

No matter if successful or not server will return the following reply:

```http
HTTP/1.1 200 OK
```

### Remove categories ###

```http
POST /command/removeCategories HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Cookie: SID=your_sid
Content-Type: application/x-www-form-urlencoded
Content-Length: length

categories=Category1%0ACategory2
```

`categories` can contain multiple cateogies separated by `\n` (%0A urlencoded)

No matter if successful or not server will return the following reply:

```http
HTTP/1.1 200 OK
```

### Set automatic torrent management ###

Requires knowing the torrent hash. You can get it from [torrent list](#get-torrent-list).

```http
POST /command/setAutoTMM HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Cookie: SID=your_sid
Content-Type: application/x-www-form-urlencoded
Content-Length: length

hashes=8c212779b4abde7c6bc608063a0d008b7e40ce32|284b83c9c7935002391129fd97f43db5d7cc2ba0&enable=true
```

`hashes` can contain multiple hashes separated by `|`<br />
`enable` is a boolean, affects the torrents listed in `hashes`, default is `false`

No matter if successful or not server will return the following reply:

```http
HTTP/1.1 200 OK
```

### Set qBittorrent preferences ###

```http
POST /command/setPreferences HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Cookie: SID=your_sid
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

For a list of possible preference options see [Get qBittorrent preferences](#get-qbittorrent-preferences)

### Get alternative speed limits state ###

Get the state of the alternative speed limits. `1` is returned if enabled, `0` otherwise.
```http
POST /command/alternativeSpeedLimitsEnabled HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Cookie: SID=your_sid
Content-Type: application/x-www-form-urlencoded
Content-Length: length
```

Server reply (example):
```http
HTTP/1.1 200 OK
content-type: text/plain
content-length: length

1
```

### Toggle alternative speed limits ###

Toggle the state of the alternative speed limits

```http
POST /command/toggleAlternativeSpeedLimits HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Cookie: SID=your_sid
Content-Type: application/x-www-form-urlencoded
Content-Length: length
```

### Toggle sequential download ###

Requires knowing the torrent hash. You can get it from [torrent list](#get-torrent-list).

```http
POST /command/toggleSequentialDownload HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Cookie: SID=your_sid
Content-Type: application/x-www-form-urlencoded
Content-Length: length

hashes=8c212779b4abde7c6bc608063a0d008b7e40ce32
```

`hashes` can contain multiple hashes separated by `|`

No matter if successful or not server will return the following reply:

```http
HTTP/1.1 200 OK
```

### Set first/last piece priority ###

Requires knowing the torrent hash. You can get it from [torrent list](#get-torrent-list).

```http
POST /command/toggleFirstLastPiecePrio HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Cookie: SID=your_sid
Content-Type: application/x-www-form-urlencoded
Content-Length: length

hashes=8c212779b4abde7c6bc608063a0d008b7e40ce32
```

`hashes` can contain multiple hashes separated by `|`

No matter if successful or not server will return the following reply:

```http
HTTP/1.1 200 OK
```

### Set force start ###

Requires knowing the torrent hash. You can get it from [torrent list](#get-torrent-list).

```http
POST /command/setForceStart HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Cookie: SID=your_sid
Content-Type: application/x-www-form-urlencoded
Content-Length: length

hashes=8c212779b4abde7c6bc608063a0d008b7e40ce32&value=true
```

`hashes` can contain multiple hashes separated by `|`<br />
`value` is a boolean, affects the torrents listed in `hashes`, default is `false`

No matter if successful or not server will return the following reply:

```http
HTTP/1.1 200 OK
```

### Set super seeding ###

Requires knowing the torrent hash. You can get it from [torrent list](#get-torrent-list).

```http
POST /command/setSuperSeeding HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Cookie: SID=your_sid
Content-Type: application/x-www-form-urlencoded
Content-Length: length

hashes=8c212779b4abde7c6bc608063a0d008b7e40ce32&value=true
```

`hashes` can contain multiple hashes separated by `|`<br />
`value` is a boolean, affects the torrents listed in `hashes`, default is `false`

No matter if successful or not server will return the following reply:

```http
HTTP/1.1 200 OK
```

### Pause all the torrents ###

Requires knowing the torrent hash. You can get it from [torrent list](#get-torrent-list).

```http
POST /command/pauseAll HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Cookie: SID=your_sid
Content-Type: application/x-www-form-urlencoded
Content-Length: length
```

### Resume all the torrents ###

Requires knowing the torrent hash. You can get it from [torrent list](#get-torrent-list).

```http
POST /command/resumeAll HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Cookie: SID=your_sid
Content-Type: application/x-www-form-urlencoded
Content-Length: length
```
