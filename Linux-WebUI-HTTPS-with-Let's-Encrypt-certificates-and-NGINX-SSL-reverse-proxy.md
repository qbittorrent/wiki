# Introduction
This is probably the easiest, most extensible and trouble-free way of setting up qBittorrent's WebUI with HTTPS. It combines ideas from these other articles of the wiki: [1][qbt-webui-https], [2][qbt-reverse-proxy].

The benefit of this setup is that with one single domain and certificate you are able to setup secure HTTPS access to various different services in your server alongside one another. For example, you may have qBittorrent's WebUI accessible at `yourdomain.com/qbt`, a simple homepage served with Apache at `yourdomain.com`, your Nextcloud instance at `yourdomain.com/nextcloud`, etc.

This guide assumes you have a working qbitorrent-nox setup (check [this][qbt-nox-wiki-setup] article if you haven't).
This guide also assumes that:
* you know how to and can forward ports on your router, to forward ports 80 and 443.
* you have setup a DNS pointing to the IP you are running the Web UI from (you can use a free one like [Duck DNS][duckdns-url]).

The overall architecture of the system will be:
```
                                 ________________________________________________
Outside world (insecure)         |   Your machine (secure)                      |
You <-------HTTPS (secure)-------|->  NGINX <----HTTP-----> qbittorrent WebUI   |
                                 |                                              |
                                 ------------------------------------------------
```

# Install the prerequisites
## Install `certbot`
[`certbot`][certbot-url] is the recommended ACME client for requesting and managing Let's Encrypt certificates. It is available on the official Ubuntu repositories, but there is an official PPA always updated with the most recent stable version, so that is the one to install. You will also need the `nginx` plugin.
```shell
sudo apt update && sudo apt upgrade -y # first update all packages in the system
sudo add-apt-repository ppa:certbot/certbot
sudo apt update
sudo apt install certbot 
sudo apt install python-certbot-nginx # this is needed for the nginx plugin
```

## Install `nginx`
You can use the version on the repositories, but if you want the most recent version you can use the PPA.
```shell
sudo apt update && sudo apt upgrade -y # first update all packages in the system
sudo add-apt-repository ppa:nginx/stable
sudo apt update
sudo apt install nginx
```
# Setup

## Setup the Web UI

1. Access your WebUI, and go to Tools -> Options -> WebUI
2. Change the following settings if they are not already like so:
* Server domains: localhost
* Port: some free port on your system that is NOT accessible through the outside world. In this case we will use `30000`
* Use UPnP / NAT-PMP to forward the port from my router: unchecked.
* Use HTTPS instead of HTTP: unchecked.


## Set up NGINX

1. Forward ports 80 and 443 in your router.
2. Allow ports 80 and 443 through your system firewall if you have one.
If you have `ufw` as your system firewall, it is as simple as:

```bash
sudo ufw allow 80 && sudo ufw allow 443 && sudo ufw reload
```

2. Clear the default files
```shell
sudo rm /etc/nginx/sites-available/*
sudo rm /etc/nginx/sites-enabled/*
```
3. Stop the `nginx` if it is running
`sudo systemctl stop nginx.service`

4. Create a config file for your reverse proxy
```shell
sudo touch /etc/nginx/sites-available/yoursite
cd /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/yoursite yoursite 
```
5. Open the file with your favourite text editor and paste something like the following (read the comments, which start with `#` to know what you have to change):
```shell
# change "yourdomain.com" and similar to your actual domain
server {
    listen 80;
    listen [::]:80;
    server_name yourdomain.com www.yourdomain.com;
    return 301 https://yourdomain.com$request_uri;
}

server {
    listen 443;
    server_name yourdomain.com www.yourdomain.com;

    # at this point we haven't created the certificate yet, but that's ok.
    # if, when creating the certificate (see below) it goes to another folder, be sure
    # to change these lines accordingly
    ssl_certificate           /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
    ssl_trusted_certificate   /etc/letsencrypt/live/yourdomain.com/chain.pem;
    ssl_certificate_key       /etc/letsencrypt/live/yourdomain.com/privkey.pem;

    ssl on;
    ssl_prefer_server_ciphers on;
    ssl_protocols TLSv1.3 TLSv1.2;
    ssl_ciphers 'ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256';
    ssl_ecdh_curve prime256v1:secp384r1:secp521r1;
    ssl_session_cache shared:TLS:50m;
    ssl_session_timeout 1d; # default is 5 min 
    ssl_session_tickets off;
    # OCSP stapling
    ssl_stapling on; 
    ssl_stapling_verify on;

    access_log            /var/log/nginx/yourdomain.access.log;

    location /qbt/ {
        # you can use any other port other than 30000 as long as it is available on your system
        proxy_pass              http://localhost:30000/;
        proxy_set_header        X-Forwarded-Host            $server_name:$server_port;
        proxy_hide_header       Referer;
        proxy_hide_header       Origin;
        proxy_set_header        Referer                     '';
        proxy_set_header        Origin                      '';
        add_header              X-Frame-Options             "SAMEORIGIN"; # see note
    }

    # OPTIONAL: serve static HTML files at the root of the domain, like a simple homepage
    location / {
        root /var/www/html;
        try_files $uri $uri/ =404;
    }

    # OPTIONAL: you can add more "location { (...) }" stanzas for other services, such as Nextcloud, etc
    #location /other_webapp {
        # change the location and port to the location and port the application is actually listening on
        #proxy_pass              http://localhost:8080/;
        #proxy_set_header        Host                        $host;
        #proxy_set_header        X-Real-IP                   $remote_addr;
        #proxy_set_header        X-Forwarded-For             $proxy_add_x_forwarded_for;
        #proxy_set_header        X-Forwarded-Proto           $scheme;
    #}
}
```

# Obtain the certificate

Run the following commands to obtain your certificate (replace `yourdomain.com` with your actual domain):
```shell
sudo certbot --nginx certonly --preferred-challenges http --must-staple --redirect --hsts --uir --staple-ocsp --rsa-key-size 4096 --domain yourdomain.com --domain www.yourdomain.com
```
Take note of the location where `certbot` stored the certificate, and adjust the nginx configuration file if needed.
If your `certbot` is setup correctly, it will renew your certificate automatically, so you do not need to worry.
You can manually test the renewal process with `sudo certbot renew --dry-run`, or actually manually renew your certificates with `sudo certbot renew`.

Note: the following five options used above are optional, but good for hardened security:
* --rsa-key-size 4096
* --must-staple
* --redirect
* --hsts
* --uir
* --staple-ocsp

Refer to the [documentation][certbot-docs-cmd-opt] for more info.

# Test your setup

Start nginx:
`sudo systemctl restart nginx.service`

Access your WebUI via `yourdomain.com/qbt`. You should see the qbittorrent Web UI and the indication that your connection is over HTTPS.


[qbt-webui-https]:https://github.com/qbittorrent/qBittorrent/wiki/Linux-Web-UI-setting-up-HTTPS-with-Let's-Encrypt-certificates
[qbt-reverse-proxy]:https://github.com/qbittorrent/qBittorrent/wiki/NGINX-Reverse-Proxy-for-Web-UI
[qbt-nox-wiki-setup]: https://github.com/qbittorrent/qBittorrent/wiki/Setting-up-qBittorrent-on-Ubuntu-server-as-daemon-with-Web-interface-(15.04-and-newer)
[duckdns-url]: https://www.duckdns.org/
[certbot-url]: https://certbot.eff.org/
[certbot-docs-cmd-opt]: https://certbot.eff.org/docs/using.html#certbot-command-line-options