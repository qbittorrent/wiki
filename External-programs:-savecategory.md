This script add Categories to **Automatically add torrents from** per **Monitored Folder**

> :point_right: This script is in flux, as it may be implemented into qBittorrent in the future.

`savecategory` expects the user's watch directories to look similar to something like this:

![](https://i.imgur.com/HBvxmUt.png)

It's based off [the popular wiki](https://old.reddit.com/r/usenet/wiki/docker) for setting up hard linking Docker and other torrent grabbers.

## Installation

Save the script below as `savecategory` and make it executable via `chmod 755 /path/to/savecategory`.

```bash
#!/bin/sh

category=$(basename $1)
torrent_hash=$2
torrent_name=$3
apiv2=http://localhost:8112/api/v2
username=admin
password=adminadmin
cookie_file=/config/external/cookie

echo "running savecategory script"

if [ ! -f "$cookie_file" ]; then
    echo "getting cookie"

    curl --silent --fail --show-error --request GET \
        --url "$apiv2/auth/login?username=$username&password=$password" \
        --cookie-jar "$cookie_file" > /dev/null
fi

echo "setting $torrent_name to category $category"

curl --silent --fail --show-error --request GET \
    --url "$apiv2/torrents/setCategory?hashes=$torrent_hash&category=$category" \
    --cookie "$cookie_file" > /dev/null

echo "completed savecategory script"

exit 0

```

> :information_source: [link to gist](https://gist.github.com/jef/e29126da5953c331310c1b6c58502be0)

And set **Run external program on torrent completion** to:

`/path/to/savecategory "%D" "%I" "%N"`

Now whenever a `.torrent` file gets dropped in your watch folders, it'll set the category correctly.