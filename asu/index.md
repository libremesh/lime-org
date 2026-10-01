
# asu misc

## auto updates asu
Run as root
```
cat << EOF > /builder/update.sh
#!/bin/bash
su - builder -c '
cd /builder
source .env
podman-compose pull
podman-compose down
podman-compose up -d
'
EOF
chmod +x /builder/update.sh

crontab -l > /tmp/crontabs
grep -qv "/builder/update.sh" /tmp/crontabs \
  || (echo "50 5 * * * /builder/update.sh" >> /tmp/crontabs \
  && crontab /tmp/crontabs)
rm /tmp/crontabs
```

## auto clean old firmware images and podman images
2026-09-30 Added a crontab to:
- clean firmware images older than 10days
- remove podman dangling images
- TODO: remove podman images of EOL openwrt release like once a month

Content of `/builder/remove_old_firmware_and_podman_images.sh`
```
#!/bin/sh

# Remove firmware images built more than 10 days ago
find /builder/public/store/ -maxdepth 1 -type d -ctime +10 -exec rm -rf {} \;

# TODO remove old images sometimes
#
# This removes only dangling images
podman image prune
```

The cronjob added (as root)
```
45 5 * * * /builder/remove_old_firmware_and_podman_images.sh
```

2026-10-01 Added a crontab to remove imagebuilders of not main/master/stable/oldstable/upcoming release once a month

Content of `/builder/remove_old_podman_images.sh`
```
#!/bin/sh

which jq || exit 0
which awk || exit 0
registry_image="ghcr.io/openwrt/imagebuilder"
# Stop on http error i.e. if the file is not a json file
versions_json=$(curl -fs -H "Accept: application/json" https://downloads.openwrt.org/.versions.json)
[ -n "$versions_json" ] || exit 0
stable_version=$(echo "$versions_json" | jq -r '.stable_version')
oldstable_version=$(echo "$versions_json" | jq -r '.oldstable_version')
upcoming_version=$(echo "$versions_json" | jq -r '.upcoming_version')

preserve="main|master|$stable_version|$oldstable_version"
if [ -n "$upcoming_version" ]; then
	preserve="$preserve|$upcoming_version"
fi
echo "Preserving versions $preserve"

su - builder -c "
cd /builder
podman images |
	grep '$registry_image' | 
	grep -Ev '$preserve' |
	awk '{ print Removing \$1 \$2 }' 

podman rmi \$(podman images |
	grep '$registry_image' |
	grep -Ev '$preserve' |
	awk '{ print \$3 }' 
)
"
```