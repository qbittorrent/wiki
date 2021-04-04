Setting up HTTPS for the web interface involves creating a key and certificate pair, and then copying the information into the corresponding fields provided by the WebUI.

The following guide assumes you have a setup as mentioned in [this](https://github.com/qbittorrent/qBittorrent/wiki/Setting-up-qBittorrent-on-Ubuntu-server-as-daemon-with-Web-interface-(14.04-and-older)) or [this](https://github.com/qbittorrent/qBittorrent/wiki/Setting-up-qBittorrent-on-Ubuntu-server-as-daemon-with-Web-interface-(15.04-and-newer)) article. Change qbtuser with the user you have qbittorrent-nox running under.


Impersonate the qbittorent user:  

`sudo su qbtuser`

Create neccesary folders:  
```
cd /home/qbtuser/
mkdir ~/.config/qBittorrent/ssl
cd ~/.config/qBittorrent/ssl
```
Now we generate the key and certificate pair:

`openssl req -new -x509 -nodes -out server.crt -keyout server.key`

Answer the questions or press enter to leave blank.


You should now have two files in your ssl folder:
```
$ ls
server.crt  server.key
```
Now go to your qbittorrent web-interface (http://192.168.0.1:8080 if you haven't changed it yet) Then open the `Tools -> Options...` in the menu bar (or click the screwdriver or cogwheel icon depending on your version), and click on the last tab labelled `WebUI`. Enable HTTPS and optionally change the port to your liking.
Then, according to your version:

- `4.2.0` and newer: copy the _path_ of the key and certificate files into the respective fields of the WebUI (for example, `/home/qbtuser/.config/qBittorrent/ssl/server.key` and `/home/qbtuser/.config/qBittorrent/ssl/server.crt`)

- older versions: copy and paste the key and certificate's _contents_ into the respective fields of the webui. You can use `cat` in your terminal to view the contents of the files:
```
cat server.key
```
Copy the contents of the entire file (including -----BEGIN PRIVATE KEY----- and -----END PRIVATE KEY-----)
into the 'key' field of the WebUI and proceed to do the same with the certificate by issuing:
```
cat server.crt
```

Now click save to have the changes take effect. 

You should now be able to log in to your qbittorrent web-server keeping in mind to change http to https and append the port number with a colon :

```
https://yourserverip:portnumber
```

A little about self signed certificates: You will be prompted by your browser that the website you are trying to reach is offering an invalid certificate. Since this is a self-signed certificate and you aren't using a third party for authenticity check, this is to be expected. Your browser is just basically telling you that it can't check the authenticity of the certificate, and because it doesn't know you created and are providing it, doesn't know that the certificate can be trusted. This is a once-off notice if you tell the browser to accept and remember the certificate, you'll only be notified again if something happens such as the certificate being changed. Your browser will proceed to download and save the certificate, and will use this next time you connect to verify and encrypt the stream.

You should now be able to log in with your username and password as before, unless you changed it during the steps above.

If you are unsure what url (ip and port and protocol) qtorrent is using, you can find it in the logfile, asuming you followed the Ubuntu Server guide on this wiki with:

`sudo tail /var/log/qbittorrent-nox.log`

Alternatively you could run qtorrent-nox in a terminal window (after impersonating the qbittorrent user) and it will print the URL to the screen with:  

`sudo su qbtuser`  
`sudo qbittorrent-nox`
