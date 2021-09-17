For users that run qBittorrent via Microsoft IIS as a reverse proxy some extra headers are needed. You must install the [URL Rewrite](https://www.iis.net/downloads/microsoft/url-rewrite) and [Application Request Routing](https://www.iis.net/downloads/microsoft/application-request-routing) addons first. Reverse proxy support will be enabled when creating the first rule.

1. In the IIS Manager, Click on the machine name to view general configuration options
2. Click on **Application Request Routing Cache**
3. On the Right hand side, Click on **Server Proxy Setting**, then tick the **Enable Proxy** box
4. Create a new site that will handle the reverse proxy requests
5. Select the site and then open **URL Rewrite**
6. On the right hand side, open **View Server Variables**
7. Click **Add** and in the box that appears enter `HTTP_X-Forwarded-Host`
8. Repeat this for `HTTP_X-Forwarded-For` and `RESPONSE_Set_Cookie`
9. Return to the rules page
10. Open **Add Rules** and select **Reverse Proxy**
11. Enter the server IP:Port without `http://` (for example `127.0.0.1:8080`), then click **OK**
12. Open the new rule and change the path to a subdirectory if needed (for example **qbweb/(.*)** = http://domain.tld/qbweb/)
13. Under **Server Variables** add the following rules:

    | Server variable name  | Value                     |
    | --------------------- | ------------------------- |
    | HTTP_X-Forwarded-Host | {HTTP_HOST}:{SERVER_PORT} |
    | HTTP_X-Forwarded-For  | {REMOTE_ADDR}             |

14. Apply and return to the rules page
15. Open **Add Rules** and select **Blank rule** under **Outbound rules**
16. Enter the name `Update Cookie Path`
17. Change **Matching scope** to `Server Variable`
18. Enter the variable name `RESPONSE_Set_Cookie`
19. Enter the pattern `^(.*; path=/)$`
20. Under **Action Properties** enter the value `{R:1}; Secure`
21. Apply

The result should look similar to this in your web.config (Note: you must use the GUI first so reverse proxy support is enabled in IIS):
```xml
            <rules>
                <rule name="Reverse Proxy">
                    <match url="qbweb/(.*)" />
                    <action type="Rewrite" url="http://127.0.0.1:8080/{R:1}" />
                    <serverVariables>
                        <set name="HTTP_X-Forwarded-Host" value="{HTTP_HOST}:{SERVER_PORT}" />
                        <set name="HTTP_X-Forwarded-For" value="{REMOTE_ADDR}" />                        
                    </serverVariables>
                </rule>
            </rules>
            <outboundRules>
                <rule name="Update Cookie Path">
                    <match serverVariable="RESPONSE_Set_Cookie" pattern="^(.*; path=/)$" negate="false" />
                    <action type="Rewrite" value="{R:1}; Secure" />
                </rule>
            </outboundRules>
```

Additionally you must untick **Enable Cross-Site Request Forgery (CSRF) protection** in qBittorrent's Web UI options for the reverse proxy to work.

You can use HTTPS to access the URL via IIS and it will use HTTP to communicate with qBittorrent. There is no need for HTTPS on localhost. 

Note: If you find yourself seeing `WebAPI login failure. Reason: IP has been banned, IP: 127.0.0.1` and needing to restart qBittorrent, you may want to set the ban after failure count to `0` which will disable it.

This tutorial is based on the assistance of Chocobo1 in [this thread](https://github.com/qbittorrent/qBittorrent/issues/7311) and in [this thread](https://github.com/qbittorrent/qBittorrent/issues/7577) for version 3.3.13 onwards.
