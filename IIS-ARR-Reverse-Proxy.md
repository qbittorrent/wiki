For users that run qBittorrent via Microsoft IIS as a reverse proxy an extra header is needed. You must install the [URL Rewrite](https://www.iis.net/downloads/microsoft/url-rewrite) and [Application Request Routing](https://www.iis.net/downloads/microsoft/application-request-routing) addons first. Reverse proxy support will be enabled when creating the first rule.

1. In the IIS Manager, Click on the machine name to view general configuration options
2. Click on **Application Request Routing Cache**
3. On the Right hand side, Click on **Server Proxy Setting**, then tick the **Enable Proxy** box
4. Create a new site that will handle the reverse proxy requests
5. Select the site and then open **URL Rewrite**
6. On the right hand side, open **View Server Variables**
7. Click **Add** and in the box that appears enter `HTTP_X-Forwarded-Host`
8. Repeat this for `HTTP_REFERER` and `HTTP_ORIGIN`
9. Return to the rules page
10. Open **Add Rules** and select **Reverse Proxy**
11. Enter the server IP:Port without `http://` (for example `127.0.0.1:8080`), then click **OK**
12. Open the new rule and change the path to a subdirectory if needed (for example **qbweb/(.*)** = http://domain.tld/qbweb/)
13. Under **Server Variables** add the following 3 rules:
```  
     Server variable name                          Value

    HTTP_X-Forwarded-Host                `{HTTP_HOST}:{SERVER_PORT}`
        HTTP_REFERER                               `0`
        HTTP_ORIGIN                                `0`
```

14. In the IIS Manager, right click on the site name and select **Explore**
15. Look for the `web.config` file and open it your text editor of choice
16. Find the lines `<set name="HTTP_REFERER" value="0" />` and `<set name="HTTP_ORIGIN" value="0" />` and delete *0* from the **value** of both.

The result should look similar to this (Note: you must use the GUI first so reverse proxy support is enabled in IIS):
```xml
<rule name="Qbittorrent" stopProcessing="true">
  <match url="qbweb/(.*)" />
  <action type="Rewrite" url="http://127.0.0.1:8080/{R:1}" />
  <serverVariables>
    <set name="HTTP_X-Forwarded-Host" value="{HTTP_HOST}:{SERVER_PORT}" />
    <set name="HTTP_REFERER" value="" />
    <set name="HTTP_ORIGIN" value="" />
  </serverVariables>
</rule>
```

You can use HTTPS to access the URL via IIS and it will use HTTP to communicate with qBittorrent. There is no need for HTTPS on localhost. 
 
This tutorial is based on the assistance of Chocobo1 in [this thread](https://github.com/qbittorrent/qBittorrent/issues/7311) and in [this thread](https://github.com/qbittorrent/qBittorrent/issues/7577) for version 3.3.13 onwards.
