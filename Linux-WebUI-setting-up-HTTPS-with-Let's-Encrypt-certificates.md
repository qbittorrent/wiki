# Introduction

You can easily setup HTTPS for the web interface with a Let's Encrypt certificate. The advantage of this method over using self-signed certificates is that all modern browsers will trust Let's Encrypt certificates by default, so you'll get no warnings when accessing the web page and there will be no need to add security exceptions.

This guide assumes you have a working qbitorrent-nox setup (check [this][qbt-nox-wiki-setup] article if you haven't).
This guide also assumes that:
* you know how to and can forward ports on your router.
* you have setup a DNS pointing to the IP you are running the Web UI from (you can use a free one like [Duck DNS][duckdns-url]).

# Install `certbot`
[`certbot`][certbot-url] is the recommended ACME client for requesting and managing Let's Encrypt certificates. It is available on the official Ubuntu repositories, but there is an official PPA always updated with the most recent stable version, so that is the one to install.
```shel
sudo apt update && sudo apt upgrade -y # first update all packages in the system
sudo add-apt-repository ppa:certbot/certbot
sudo apt update
sudo apt install certbot 
```

# Obtaining the certificate

You will need either port 80 or 443 free during this process. If you have additional software running on your machine bound to port 443, you will want to use port 80 and vice-versa. **In this example we will use port 80 for the certificate issuance process**. If you want to use port 443, the only thing that you need to change besides the port number would be the `--preferred-challenges` from `http` to `tls-sni` in the command below. Refer to the cerbot documentation for more information on this.

The method used for obtaining the certificate will be the `standalone` method. Before you actually run the necessary command, you need to:
* temporarily stop any program/server bound to port 80 (check with `sudo netstat -tulpn`, for example);
* forward port 80 on your router;
* allow port 80 on your firewall (most likely `ufw`) if it is not already allowed;

Now, run the following commands to obtain your certificate (replace `yourwebuidomain.duckdns.org` with your actual domain):
```shell
sudo ufw allow 80
sudo ufw reload
sudo certbot certonly --standalone --preferred-challenges http --must-staple --redirect --hsts --uir --staple-ocsp --rsa-key-size 4096 --domain yourwebuidomain.duckdns.org
```
After the certificate is generated successfully, you can restart any program you had listening on port 80 or maybe re-block port 80 if you weren't using it (`sudo ufw deny 80 && sudo ufw reload`).

Note: the following five options used above are optional, but good for hardened security:
* --rsa-key-size 4096
* --must-staple
* --redirect
* --hsts
* --uir
* --staple-ocsp

Refer to the [documentation][certbot-docs-cmd-opt] for more info

# Installing the certificate on the Web UI

1. Go to your Web UI, `yourwebuidomain.duckdns.org`.
2. On the Tools -> Options... menu, go to the Web UI tab.
3. In the "Server domains:" field put `yourwebuidomain.duckdns.org`
4. Tick the "Use HTTPS instead of HTTP" checkbox
5.
    - If using version `4.2.0` or later:
        - In the "Key:" text box paste the _path_ of the key file.
        - In the "Certificate:" text box paste the _path_ of the certificate file.
        - **Important Note**: since the directory where these files are located (for example, `/etc/letsencrypt/live/yourwebuidomain.duckdns.org/`) is usually only readable by `root`, you may first need to copy the files somewhere that is readable by the user account that qBittorrent is running under. Do not change the permissions of the original `certbot` directories.

    - If using older versions:
        - In the "Key:" text box paste the _contents_ of the key file (for example, `/etc/letsencrypt/live/yourwebuidomain.duckdns.org/privkey.pem`). You may need root privileges to access this file.
        - In the "Certificate:" text box paste the _contents_ of the certificate file (for example, `/etc/letsencrypt/live/yourwebuidomain.duckdns.org/fullchain.pem`). You may need root privileges to access this file.

6. Click save, close the tab and now you should only be able to access your Web UI via HTTPS.

# Automating certificate renewal

Your certificates expire after 90 days, but you can renew them manually or setup automatic renewal for your certificates.

A possible renewal command for a user that does not normally use port 80 can be:

`sudo certbot renew --pre-hook "ufw allow 80 && ufw reload" --post-hook "ufw deny 80 && ufw reload"`

If you have a program listening on port 80, be sure to use the `--pre-hook` and `--post-hook` flags to restart it (for example, `--pre-hook "stop_my_program.sh"` and `--post-hook "restart_my_program.sh"`).

Additionally, you can use `certbot` hooks to copy certificate files around and even to shutdown/restart qBittorrent and possibly even modify its config.

Each time the command is run, `certbot` checks if any certificate is more than 60 days old, and only actually renews those.

You can put your renewal command (without `sudo`) in a crontab or a systemd unit set to run daily or twice a day, which is what the Let's Encrypt folks recommend.


[qbt-nox-wiki-setup]: https://github.com/qbittorrent/qBittorrent/wiki/Setting-up-qBittorrent-on-Ubuntu-server-as-daemon-with-Web-interface-(15.04-and-newer)
[duckdns-url]: https://www.duckdns.org/
[certbot-url]: https://certbot.eff.org/
[certbot-docs-cmd-opt]: https://certbot.eff.org/docs/using.html#certbot-command-line-options