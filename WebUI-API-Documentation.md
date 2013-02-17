**This is a work in progress, it's still not finished**

# Table of Contents #

1. [Authorization](#auth)
1. [GET methods](#GET)
  1. [Shutdown qBittorrent](#shutdown)
  1. [Get torrent list](#torrentlist)
  1. [Get torrent generic properties](#propsgen)
1. POST methods
  1. Download torrent from URL
  1. Upload torrent from disk
1. Unsorted/Currently unknown
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
  1. Get preferences ?json/preferences?
  1. Get transfer info (status bar) ?json/transferInfo?

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
Authorization: <your_auth_string>
```

### <a id="torrentlist"></a>Get torrent list ###

```
GET /json/torrents HTTP/1.1
User-Agent: Fiddler
Host: 127.0.0.1
Authorization: <your_auth_string>
```
Server will return the following one-line comma-separated string (example):

```
[{"hash":"0283b35cc14387a4f6d01de33aa012fb1b740df6","name":"Sword of the Stars II Enhanced Edition","size":"1.8 ГиБ","progress":1,"dlspeed":"0 Б/с","upspeed":"0 Б/с","priority":"*","num_seeds":"0","num_leechs":"0","ratio":"0.1","eta":"∞","state":"stalledUP"},{<another_torrent_info>}]
```
where

`hash` - torrent hash; most queries will use torrent hash as parameter<br/>
`name` - torrent name<br/>
`size` - size of files and folders in torrent selected for download<br/>
`progress` - float number for download completion percentage, where 1 == 100%, 0 == 0%, and, 0.58 == 58%<br/>
`dlspeed` and `upspeed`: torrent download and upload speed respectively<br/>
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

`state` - possible values:<br/>

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

### <a id="propsgen"></a>Get torrent generic properties ###