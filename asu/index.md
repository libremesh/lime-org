
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