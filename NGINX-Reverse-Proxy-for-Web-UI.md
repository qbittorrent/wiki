This configuration allows you to use NGINX as a reverse proxy for the WebUI listening on a local address to expose it outside of your LAN, on the Web. It is assumed that your WebUI is configured to be accessible at `http://127.0.0.1:30000/`, and you wish to be able to access it outside of your LAN at `mywebsite.com/qbt`. Then, in the NGINX configuration used to serve `mywebsite.com`, your `location /qbt/` stanza should have the following settings:

```nginx
location /qbt/ {
    proxy_pass         http://127.0.0.1:30000/;
    proxy_http_version 1.1;

    proxy_set_header   Host               127.0.0.1:30000;
    proxy_set_header   X-Forwarded-Host   $http_host;
    proxy_set_header   X-Forwarded-For    $remote_addr;

    # not used by qBittorrent
    #proxy_set_header   X-Forwarded-Proto  $scheme;
    #proxy_set_header   X-Real-IP          $remote_addr;

    # optionally, you can adjust the POST request size limit, to allow adding a lot of torrents at once
    #client_max_body_size 100M;

    # Since v4.2.2, is possible to configure qBittorrent
    # to set the "Secure" flag for the session cookie automatically.
    # However, that option does nothing unless using qBittorrent's built-in HTTPS functionality.
    # For this use case, where qBittorrent itself is using plain HTTP
    # (and regardless of whether or not the external website uses HTTPS),
    # the flag must be set here, in the proxy configuration itself.
    # Note: If this flag is set while the external website uses only HTTP, this will cause
    # the login mechanism to not work without any apparent errors in console/network resulting in "auth loops".
    proxy_cookie_path  /                  "/; Secure";
}
```

Note: If you find yourself seeing `WebAPI login failure. Reason: IP has been banned, IP: 127.0.0.1` and needing to restart qBittorrent, you may want to set the ban after failure count to `0` which will disable it.

---

Obsolete directives, no longer needed when using recent qBittorrent versions

- No longer required/discouraged since v4.1.2:

    ```nginx
    # The following directives effectively nullify Cross-site request forgery (CSRF)
    # protection mechanism in qBittorrent, only use them when you encountered connection problems.
    # You should consider disable "Enable Cross-site request forgery (CSRF) protection"
    # setting in qBittorrent instead of using these directives to tamper the headers.
    # The setting is located under "Options -> WebUI tab" in qBittorrent since v4.1.2.
    proxy_hide_header Referer;
    proxy_hide_header Origin;
    proxy_set_header  Referer  '';
    proxy_set_header  Origin   '';
    ```

- No longer required since v4.1.0:

    ```nginx
    add_header X-Frame-Options "SAMEORIGIN";
    ```