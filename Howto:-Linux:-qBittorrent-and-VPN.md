## Subject:
If you want to use the qBittorrent together with a VPN connection for any reason (to maintain your privacy, to avoid your ISP's restrictions or to enable incoming connections without paying for a static IP, or all those reasons simultaneously), you can set up your Linux system like following.

## Requirements:
The control over the OpenVPN server deployed on the VPS with a static worldwide-routable IP address ("white IP address"), the rest will be about setting up the OpenVPN+qBittorrent, but OpenVPN is not the only VPN solution and just one of the possible solutions.

## Pre-requirements:
Before the howto itself, let's assume you have installed the qBittorrent on your device from the [official PPA](http://ppa.launchpad.net/qbittorrent-team/qbittorrent-stable/ubuntu) or from the packages downloaded from the [official site](https://www.qbittorrent.org/download.php) and set up your VPN connection and checked its connectivity on the device intended to host the qBittorrent client.

You also may want to add the following line to your OpenVPN client configuration file to avoid it becoming the default gateway:
```
pull-filter ignore redirect-gateway
```
This line will allow all traffic not intended to go through the VPN to go through the primary ISP gateway.

## The task should be considered in two parts:
1. Setting up the qBittorrent client to work through a VPN connection
2. Enabling incoming connections from outer space to the qBittorrent through the VPN.

Let's assume you have the VPN connection interface named `tun0`, VPN server external IP is 212.213.214.215, VPN gateway IP is `10.8.0.1/24` and VPN Client IP address is `10.8.0.2/24` - feel free to do replace any of those values in the guide below if it does not match your setup.

## Part 1:
Setting up the qBittorrent client to work through a VPN connection.

### 0. Make sure qBittorrent has the CAP_NET_RAW capability.
If you are using qbittorrent-nox - verify its systemd unit has the following line in the `[Service]` section:
```
AmbientCapabilities=CAP_NET_RAW
```
If you are using the qBittorrent with the GUI or don't use systemd - use a proper way to gain the client `CAP_NET_RAW` capability, or just run it as `root` user (not recommended).

### 1. Set up qBittorrent to bind to your VPN connection.
 Add the following lines to your qBittorrent.conf into the `[Preferences]` section:
```
Connection\Interface=tun0
Connection\InterfaceAddress=10.8.0.2
```
This also can be performed on the Web GUI Settings page or X11 GUI window - just look "Advanced" settings page for "Network interface" and "Optional IP address to bind to" and set up those options there.

### 2. Create an auxiliary routing table.
Edit the `/etc/iproute2/rt_tables` and add the following line to this file:
```
200 isp2
```
isp2 is a routing table name, it can be arbitrary. 200 is the priority of this routing table, it should be less than the default routing table priority (253 default).

### 3. Set up auxiliary routing rules:
Execute the following command:
```
ip rule add from 10.8.0.2 table isp2 prio 1
```
Here the `from 10.8.0.2 table isp2 prio 1` means all traffic with a source IP address 10.8.0.2 will be processed using routing table isp2 with priority 1

### 4. Filling up auxiliary routing table:
Execute the following command:
```
ip route add default via 10.8.0.1 dev tun0 table isp2
```
It will add a default route through the VPN gateway using device tun0 to the table isp2.

After performing steps 0..4 try to use your qBittorrent instance to download anything and make sure it using only the VPN interface for peers connection - it should now use only tun0 interface and stop if this interface will become unavailable (like if the OpenVPN daemon gets down or OpenVPN connection gets disconnected).

## Part 2: 
Set up the VPN server and VPN client to accept incoming connections from outer space.

### 0. Set up a static port for peers connections:
Add the following line to your qBittorrent.conf to the `[Preferences]` section:
```
Connection\PortRangeMin=62000
```
This also can be performed on the Web GUI Settings page or X11 GUI window - just look "Connection" settings page for "Port used for incoming connections" and set up this option there.
It's better to use the port in the dynamic port range (49152 to 65535) as some ISPs may throttle traffic for lower ports.


### 1. Set up the OpenVPN server to forward incoming connections to the VPN client:
Enable forwarding from incoming interface `eth0` to VPN interface `tun0` using the following command:
```
iptables -t filter -A FORWARD -i eth0 -o tun0 -j ACCEPT
```
Then forward desired TCP and UDP port to yours VPN client IP address using following commands:
```
iptables -t nat -A PREROUTING -d 212.213.214.215/32 -p tcp -m tcp --dport 62000 -j DNAT --to-destination 10.8.0.2:62000
iptables -t nat -A PREROUTING -d 212.213.214.215/32 -p udp -m udp --dport 62000 -j DNAT --to-destination 10.8.0.2:62000
iptables -t nat -A POSTROUTING -d 10.8.0.2/32 -p tcp -m tcp --sport 62000 -j SNAT --to-source 212.213.214.215:62000
iptables -t nat -A POSTROUTING -d 10.8.0.2/32 -p udp -m udp --sport 62000 -j SNAT --to-source 212.213.214.215:62000
```
Please don't forget to add iptables commands on the OpenVPN server or save iptables rules using the following command:
```
iptables-save > /etc/iptables/rules.v4
```
Restart the qBittorrent client and check does it detects incoming connections possibility or not (please wait for some time - it can detect connection changes with a delay).

If qBittorrent does not receive incoming connections - proceed to the following steps:

### 2. Temporarily disable rp_filter:
This may be necessary to allow packages that don't have the proper return route to be processed on your system.
Execute the following command:
```
for i in /proc/sys/net/ipv4/conf/*/rp_filter ; do echo 0 > $i ; done
```
This will disable rp_filter on all interfaces just for this session, after reboot your device will restore its default settings for rp_filter.

After disabling rp_filter please re-check qBittorrent - it should detect incoming connections possibility and should be able to seed using the `tun0` interface.

After doing the checks above you can disable rp_filter permanently if necessary.

### 3. Disable rp_filter permanently:
Execute the following commands:
```
sed -i 's/net.ipv4.conf.default.rp_filter=2/net.ipv4.conf.default.rp_filter=0/g' /etc/sysctl.d/10-network-security.conf
sed -i 's/net.ipv4.conf.all.rp_filter=2/net.ipv4.conf.all.rp_filter=0/g' /etc/sysctl.d/10-network-security.conf
```
This should prevent rp_filter from enabling after the client's device reboots.

That's all.
