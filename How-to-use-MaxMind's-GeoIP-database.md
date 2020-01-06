Up until December 30th 2019 Maxmind offered the GeoIP-country database via a public URL. But after California Consumer Privacy Act (CCPA) it decided to change its distribution strategy. More details [here](https://blog.maxmind.com/2019/12/18/significant-changes-to-accessing-and-using-geolite2-databases/).

Now, in order to automatically download and use the GeoIP-country database with qBittorrent, you have to [create a free account](https://www.maxmind.com/en/geolite2/signup) with MaxMind and get a free license key. **This is available from v4.2.3 onwards.**

If you enable `Resolve peer countries (GeoIP)` in the Advanced Settings you have 2 options:
1. You can put your license key in `GeoIP license key` and have qbittorrent automatically download the database on a monthly period
2. Or you can leave the license key field empty and provide your own database file. You may have obtained the file from other links/sources. The database filename must be `GeoLite2-Country.mmdb` and it must be placed under the `GeoIP` folder located in the same folder where `BT_backup` folder resides.