If you're using NGINX as a reverse proxy for Web UI, as of version 3.3.14 (3.3.13 will break your setup), you'll need to add this to your `location /`


```
        proxy_hide_header   Referer;
        proxy_hide_header   Origin;
        proxy_set_header    Referer                 '';
        proxy_set_header    Origin                  '';
```