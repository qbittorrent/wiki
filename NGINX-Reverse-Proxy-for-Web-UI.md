If you're using NGINX as a reverse proxy for Web UI, as of version 3.3.15, assuming your reverse proxy is local, your `location /` should look similar to the following:


```
    location / { 
...
        proxy_pass          http://127.0.0.1:<port>/;
...
        proxy_set_header    X-Forwarded-Host        127.0.0.1:<port>;
...
        proxy_hide_header   Referer;
        proxy_hide_header   Origin;
        proxy_set_header    Referer                 ''; 
        proxy_set_header    Origin                  ''; 
...
    }
```