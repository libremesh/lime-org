
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
