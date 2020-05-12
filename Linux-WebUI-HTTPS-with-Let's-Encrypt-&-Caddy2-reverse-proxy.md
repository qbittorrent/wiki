# Introduction
A very straightforward & easy way to add HTTPS to your FQDN pointing to qbittorrent.
This guide assumes you have a working qbitorrent setup as well as:
* you know how to and can forward ports on your router, to forward ports 80 and 443.
* you have setup a FQDN pointing to the IP you are running the Web UI from.

The overall architecture of the system will be:
```
                                 ________________________________________________
Outside world (insecure)         |   Your machine (secure)                      |
You <-------HTTPS (secure)-------|->  Caddy2 <----HTTP-----> qbittorrent WebUI  |
                                 |                                              |
                                 ------------------------------------------------
```

## Install Caddy2
On arch based distros that's as easy as
```
yay -S caddy2
```

# Setup
## Activate WebUI in qbittorrent
1. Access your WebUI, and go to Tools -> Options -> WebUI
2. Make note of the port specified, leave your IP set to *
3. Deselect "Use UPnP / NAT-PMP to forward the port from my router."
4. Deselect "Use HTTPS instead of HTTP."
5. Enable clickjacking protection
6. Enable CSRF protection
7. Enable host header validaion. Confirm "*; example.domain" is in the "server domains" text box. 

## Caddy2 Config
Create a Caddyfile as a config. Assuming a standard HTTPS & reverse proxy setup, your Caddyfile can be as basic as
```
{
    email example@email.com
}

example.domain {
    reverse_proxy IP:port 
}

```

## Run Caddy
Forward ports 80 and 443 in your router.
Allow ports 80 and 443 through your system firewall if you have one.
Run one of the following commands
```
sudo caddy run -config /path/to/Caddyfile
```
or
```
sudo caddy start -config /path/to/Caddyfile
```
The difference between the two is minimal.
run starts the Caddy process and blocks indefinitely while start starts the Caddy process in the background and then returns. 
You can also run caddy as a systemd service.

# Test
Open up your favorite browser and enter your FQDN into the URL bar. You should see the qbittorrent Web UI and the indication that your connection is over HTTPS.