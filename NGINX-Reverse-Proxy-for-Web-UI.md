If you're using NGINX as a reverse proxy for Web UI, as of version 3.3.15, assuming your reverse proxy is local, your `location /` should have the following settings:

```nginx
location / {
    proxy_set_header   X-Forwarded-Host  $host:$server_port;
    proxy_hide_header  Referer;
    proxy_hide_header  Origin;
    proxy_set_header   Referer           '';
    proxy_set_header   Origin            '';
}
```
For some users, several windows in the Web UI will still be blank, such as when adding a new torrent from a URL/magnet or local file. If so, try adding the following line to the location block:

`add_header  X-Frame-Options "SAMEORIGIN";`