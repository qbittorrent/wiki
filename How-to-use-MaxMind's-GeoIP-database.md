**NOTE:** Current versions of qBittorrent rely on the [db-ip Country Lite](https://db-ip.com/db/download/ip-to-country-lite) database, which will be downloaded automatically during startup.

If for some reason you require manual installation:

1. Download the MMDB format from the above website (accept conditions)
2. Create a `GeoDB` folder in the folder where `BT_backup` is located
3. Unpack the downloaded archive into the `GeoDB` folder, and rename database file to `dbip-country-lite.mmdb`

***

Up until December 30th 2019 Maxmind offered the GeoIP-country database via a public URL. But after California Consumer Privacy Act (CCPA) it decided to change its distribution strategy. More details [here](https://blog.maxmind.com/2019/12/18/significant-changes-to-accessing-and-using-geolite2-databases/).

Now, in order to automatically download and use the GeoIP-country database with qBittorrent, you have to [create a free account](https://www.maxmind.com/en/geolite2/signup) with MaxMind and get a free license key. **This is available from v4.2.2 onwards.**

If you enable `Resolve peer countries (GeoIP)` in the Advanced Settings you have 2 options:
1. You can put your license key in `GeoIP license key` and have qbittorrent automatically download the database on a monthly period
2. Or you can leave the license key field empty and provide your own database file. You may have obtained the file from other links/sources. The database filename must be `GeoLite2-Country.mmdb` and it must be placed under the `GeoIP` folder located in the same folder where `BT_backup` folder resides.