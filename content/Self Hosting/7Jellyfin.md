---
title: 7. Jellyfin
tags:
  - Docker
  - Portainer
  - Cloudflare
  - NGINX
  - SelfHosting
  - Server
  - Media
  - Sonarr
  - Radarr
  - Containers
  - ReverseProxy
---
# Preamble

One of the things that makes me the angriest is the state of video streaming right now. If I want to watch spongebob from start to finish, you need at least 4 different streaming services. This is not fun for me so my brother showed me how his jellyfin works. He showed me the entire process and workflow of how to add things onto jellyfin! So I'm going to make my own jellyfin and get a couple storage drives to stream some shows for my friends and I!
# Implementation

## Jellyfin

1. Run the Jellyfin container
```bash
docker run -d \
  --name jellyfin \
  -p 8096:8096 \
  -v jellyfin_config:/config \
  -v jellyfin_cache:/cache \
  --restart=always \
  jellyfin/jellyfin
```

## NGINX

1. Add an NGINX site config
```bash
sudo nano /etc/nginx/sites-available/jellyfin
```

jellyfin
```nginx
server {
    listen 80;
    server_name jellyfin.loganharmondeveloper.com;

    location / {
        proxy_pass http://localhost:8096;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /socket {
        proxy_pass http://localhost:8096/socket;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

2. Symbolic link and reload
```bash
sudo ln -s /etc/nginx/sites-available/jellyfin /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

## Cloudflare

1. Add to Cloudflare tunnel
```bash
sudo nano /etc/cloudflared/config.yml
```

config.yml
```yaml
  - hostname: jellyfin.loganharmondeveloper.com
    service: http://localhost
```

2. Restart cloudflared
```bash
sudo systemctl restart cloudflared
```
2. Add the DNS record
```bash
cloudflared tunnel route dns pterodactyl-tunnel jellyfin.loganharmondeveloper.com
```

Jellyfin is now up and running on jellyfin.loganharmondeveloper.com!

## New SSD

I got this SSD from my brother exactly for this purpose so now I'm going to put it to use. My disk wasn't partitioned so I need to format and partition it before I do anything.

1. Plug in SSD and check if linux can see it
```bash
lsblk
```

Mine was called "sdb -> sdb1" but it may be different if there is multiple things plugged in.

2. Launch fdisk to create a partition for the SSD
```bash
sudo fdisk /dev/sdb
```

Follow these commands
n -> p -> 1 -> enter -> enter -> w

3. Wipe the partition clean
```bash
sudo wipefs -a /dev/sdb1
```

4. Format to ext4
```bash
sudo mkfs.ext4 /dev/sdb1
```

5. Mount the drive
```bash
sudo mkdir -p /mnt/media
sudo mount /dev/sdb1 /mnt/media
sudo chown -R 1000:1000 /mnt/media
```

6. Check that it's mounted
```bash
df -h /mnt/media
```

```bash
docker stop jellyfin
docker rm jellyfin

docker run -d \
  --name jellyfin \
  -p 8096:8096 \
  -v jellyfin_config:/config \
  -v jellyfin_cache:/cache \
  -v /mnt/media:/media \
  --restart=always \
  jellyfin/jellyfin
```

7. Check that the folders are in the jellyfin container
```bash
docker exec -it jellyfin ls /media
```

At this point I made the jellyfin admin account in the webpage and I created the media folders for movies and shows. Next I'm going to move onto deploying Sonarr and Radarr with docker for my media management. Now I don't want Jellyfin to be running like how it is now so I'm going to put it in the media-stack docker-compose file in the next note. 

8. Stop the jellyfin container
```bash
docker stop jellyfin
docker rm jellyfin
```

The next note will be all about the usenet!