This script add Categories to **Automatically add torrents from** per **Monitored Folder**

> :point_right: This script is in flux, as it may be implemented into qBittorrent in the future.

`savecategory` expects the user's watch directories to look similar to something like this:

![](https://i.imgur.com/HBvxmUt.png)

It's based off [the popular wiki](https://old.reddit.com/r/usenet/wiki/docker) for setting up hard linking Docker and other torrent grabbers.

## Installation

Save the script below as `savecategory` and make it executable via `chmod 755 /path/to/savecategory`.

```bash
#!/bin/sh

category="$(basename $1)"
torrent_hash="$2"
torrent_name="$3"
host="http://localhost:8112"
username="admin"
password="adminadmin"

echo "running savecategory script"

echo "\tgetting cookie"

cookie=$(curl --silent --fail --show-error \
    --header "Referer: $host" \
    --cookie-jar - \
    --request GET "$host/api/v2/auth/login?username=$username&password=$password")

echo "\tsetting $torrent_name to category $category"

echo "$cookie" | curl --silent --fail --show-error \
    --cookie - \
    --request GET "$host/api/v2/torrents/setCategory?hashes=$torrent_hash&category=$category"

echo "completed savecategory script"

exit 0

```

> :information_source: Make sure to replace the `username` and `password` with your credentials before using or else this will not work. Another caveat is that if your password contains `#` or `&`, you'll need to replace with [ASCII encoded characters](https://www.w3schools.com/tags/ref_urlencode.ASP).
> 
> :link: [gist](https://gist.github.com/jef/e29126da5953c331310c1b6c58502be0) for potential script changes or comments.

And set **Run external program on torrent completion** to:

`/path/to/savecategory "%D" "%I" "%N"`

On completion, the category will change based on the directory name the `.torrent` file was placed in the watch directory.