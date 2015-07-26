Setting up HTTPS for the web interface involves creating a key and certificate pair, and then copying the information into the corresponding fields provided by the WebUI.

The following guide assumes you have a setup as mentioned in [this](Setting-up-qBittorrent-on-Ubuntu-server-as-daemon-with-Web-interface) article. Change qbtuser with the user you have qbittorrent-nox running under.

First we create the folders:
```
cd /home/qbtuser/
mkdir ~/.config/qBittorrent/ssl
cd ~/.config/qBittorrent/ssl
```
Now we generate the key file:

`openssl genrsa -des3 -out qbittorrent.key 1024`

This will start creating a key file. You will be asked for a phrase used to generate the key. Whatever it is, make sure you can type it in twice in succession, and then again when you generate the certificate. Some tips: I wouldn't bother trying to remember the phrase in the long term. If you were to lose this phrase, you could always delete the certificates and generate a new pair.

Now lets proceed with the certificate file:

`openssl req -new -key qbittorrent.key -out qbittorrent.cert`

You will now be asked to enter the pass phrase for qbittorrent.key. Use the phrase you used when creating the key file above and press ENTER. Press enter for all the successive prompts, or fill them in if you prefer.

You should now have two files in your ssl folder:
```
$ ls
qbittorrent.cert  qbittorrent.key
```
Now go to your qbittorrent web-interface (http://192.168.0.1:8080 if you haven't changed it yet) Then open options (look for gear and screwdriver icon), and click on the last tab labelled WebUI. Enable HTTPS change the port to your liking. Now in your terminal, were going to copy and paste the key and certificate's contents into the respective fields of the webui.
```
cd /home/qbtuser/.config/qBittorrent/ssl/
sudo nano qbittorrent.key
```
Copy the contents of the entire file into the 'key' field of the WebUI. Now do

`sudo nano qbittorrent.cert`

and copy the entire file contents to the 'certificate' field. Scroll down and edit the username and password fields to your liking (leaving them the default omits the implementation of HTTPS/SSL. Now, little further down, click save. You can now restart the server in the terminal with:

`sudo service qbittorrent restart`

to have the changes take effect. 

You should now be able to log in to your qtorrent web-server at the url you specified for example https://192.168.0.1:54343. 

A little about self signed certificates: You will be prompted by your browser that the website you are trying to reach is offering an invalid certificate. Since this is a self-signed certificate and you aren't using a third party for authenticity check, this is to be expected. Your browser is just basically telling you that it can't check the authenticity of the certificate, and because it doesn't know you created and are providing it, doesn't know that the certificate can be trusted. This is a once-off notice if you tell the browser to just ignore and accept the certificate. Your browser will proceed to download and save the certificate, and will use this next time you connect to verify and encrypt the stream.

You should now be able to log in with your username and password.

If you are unsure what url (ip and port and protocol) qtorrent is using, you can find it in the logfile, asuming you followed the Ubuntu Server guide on this wiki with:

`sudo tail /var/log/qbittorrent-nox.log`

Alternatively you could run qtorrent-nox in a terminal window and it will print the URL to the screen with:

`sudo qbittorrent-nox`

--------

This guide failed to create usable certs because of the passphrase requirement of `genrsa`. I suggest the following amendment `openssl req -new -x509 -nodes -out server.crt -keyout server.key` for ssl key and cert creation. -CS