# Introduction

You can easily setup HTTPS for the web interface with a Let's Encrypt certificate. The advantage of this method over using self-signed certificates is that all modern browsers will trust Let's Encrypt certificates by default, so you'll get no warnings when accessing the web page and there will be no need to add security exceptions.

This guide assumes you have a working qbitorrent-nox setup (check [this](https://github.com/qbittorrent/qBittorrent/wiki/Setting-up-qBittorrent-on-Ubuntu-server-as-daemon-with-Web-interface-(15.04-and-newer)) article if you haven't).
This guide also assumes that:
* you know how to and can forward ports on your router.
* you have setup a DNS pointing to the IP you are running the Web UI from (you can use a free one like [Duck DNS](https://www.duckdns.org/)).

# Install `certbot`
[`certbot`](https://certbot.eff.org/) is the recommended ACME client for requesting and managing Let's Encrypt certificates. It is available on the official Ubuntu repositories, but there is an official PPA always updated with the most recent stable version, so that is the one to install.
```shel
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository ppa:certbot/certbot
sudo apt update
sudo apt install certbot 
```

# Obtaining the certificate

The method used for obtaining the certificate will be the `standalone` method. As such, before you actually run the necessary command, you need to:
* temporarily stop any program/server listening on port 80 (check with `sudo netstat -tulpn`, for example)
* forward port 80 on your router
* allow port 80 on your firewall (most likely `ufw`) if it is not already allowed

So, after forwarding port 80 and temporarily stopping any program listening on port 80, run the following commands to obtain your certificate (replace `yourwebuidomain.duckdns.org` with your actual domain):
```shell
sudo ufw allow 80
sudo ufw reload
sudo certbot certonly --standalone -d yourwebuidomain.duckdns.org
```
After the certificate is generated successfully, you can restart any program you had listening on port 80 or maybe re-block port 80 if you weren't using it (`sudo ufw deny 80 && sudo ufw reload`).

# Installing the certificate on the Web UI

1. Go to your Web UI, `yourwebuidomain.duckdns.org`.
2. On the Tools -> Options... menu, go to the Web UI tab.
3. In the "Server domains:" field put `yourwebuidomain.duckdns.org`
4. Tick the "Use HTTPS instead of HTTP" checkbox
5. In the "Key:" text box paste the contents of the file `/etc/letsencrypt/live/yourwebuidomain.duckdns.org/privkey.pem`. You may need root privileges to access this file.
6. In the "Certificate:" text box paste the contents of the file `/etc/letsencrypt/live/yourwebuidomain.duckdns.org/fullchain.pem`. You may need root privileges to access this file.
7. Click save, close the tab and now you should only be able to access your Web UI via HTTPS.

# Automating certificate renewal

Your certificates expire after 90 days, but you can renew them manually or setup automatic renewal for your certificates.

A possible renewal command for a user that does not normally use port 80 can be:

`sudo certbot renew --pre-hook "ufw allow 80 && ufw reload" --post-hook "ufw deny 80 && ufw reload"`

If you have a program listening on port 80, be sure to use the `--pre-hook` and `--post-hook` flags to restart it (for example, `--pre-hook "stop_my_program.sh"` and `--post-hook "restart_my_program.sh"`).

Each time the command is run, `certbot` checks if any certificate is more than 60 days old, and only actually renews those.

You can put your renewal command (without `sudo`) in a crontab or a systemd unit set to run daily or twice a day, which is what the Let's Encrypt folks recommend.

# Note on the port used for `certbot`

By default, `certbot` uses port 80 for challenges. I find this default to be convenient, since, even though I run webservers other that the qBittorrent WebUI on my machine, those use HTTPS and listen on port 443 exclusively.

When I wrote this I assumed most people either don't have additional web servers or if they do, they aren't actually using port 80 (and they might actively block it). Thus, for most users there is no need restart anything when obtaining or renewing the certificate for the qBittorrent Web UI, when following this tutorial verbatim.

If, by chance, you happen to have a web server or other program listening on port 80 and your port 443 is free, you can instruct `certbot` to use port 443 for the challenges (see the documentation), thus eschewing the need to restart programs listening on port 80 when obtaining/renewing the certificate for the qBittorrent Web UI.