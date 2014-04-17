Setting up HTTPS for the web interface involves creating a key and certificate pair, and then copying the information into the corresponding fields provided by the WebUI.
```
cd /home/qbtuser/
mkdir ~/.config/qBittorrent/ssl
cd ~/.config/qBittorrent/ssl
openssl genrsa -des3 -out qbittorrent.key 1024
```

This will start creating a key file. You will be asked for a phrase used to generate the key. Whatever it is, make sure you can type it in twice in succession, and then again when you generate the certificate. Some tips: I wouldn't bother trying to remember the phrase in the long term. If you were to lose this phrase, you could always delete the certificates and generate a new pair.

Now lets proceed with the certificate:

openssl req -new -key qbittorrent.key -out qbittorrent.cert

You will now be asked to: Enter pass phrase for qbittorrent.key:. Type in the key you used to create the key above and hit ENTER. Press enter for all the successive prompts.

You should now have two files in your ssl folder:

$ ls
qbittorrent.cert  qbittorrent.key

Now go to your qbittorrent web-interface (http://192.168.0.1:8080 if you haven't changed it yet) Then open options (look for gear and screwdriver icon), and click on the last tab labelled WebUI. Enable HTTPS change the port to your liking. Now in your terminal, were going to copy and paste the key and certificate's contents into the respective fields of the webui.

cd /home/qbtuser/.config/qBittorrent/ssl/
sudo nano qbittorrent.key

Copy the contents of the entire file into the 'key' field of the WebUI. Now do

sudo nano qbittorrent.cert

and copy the entire file contents to the 'certificate' field. Scroll down and edit the username and password fields to your liking.

Now little further, click save. You can now restart the server in the terminal with:

sudo service qbittorrent restart

to have the changes take effect.