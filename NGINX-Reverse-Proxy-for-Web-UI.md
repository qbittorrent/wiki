If you're using NGINX as a reverse proxy for Web UI, as of version 4.0.3, assuming your reverse proxy is local, your `location /qbt/` should have the following settings:

```nginx
location /qbt/ {
    proxy_pass              http://127.0.0.1:8080/;
    proxy_set_header        X-Forwarded-Host        $server_name:$server_port;
    proxy_hide_header       Referer;
    proxy_hide_header       Origin;
    proxy_set_header        Referer                 '';
    proxy_set_header        Origin                  '';
    # add_header              X-Frame-Options         "SAMEORIGIN"; # not needed since 4.1.0
}
```
