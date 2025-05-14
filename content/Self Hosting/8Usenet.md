---
title: 8. Usenet
tags:
  - Usenet
  - Docker
  - Jellyfin
  - Cloudflare
  - NGINX
  - ReverseProxy
  - Sonarr
  - Radarr
  - SelfHosting
  - SABnzbd
  - Indexer
  - UsenetProvider
  - Newshosting
  - NZBGeek
  - DrunkenSlug
  - Media
---
# Preamble

So if the reader is curious at all, the usenet is completely legal. So all the setup and the process for this is good, the only thing that is a shady is downloading copyrighted content. That is still considered under DMCA and it's possible to get a penalty from downloading copyrighted content. Thats the only thing that a person using the usenet should be wary for.

So if you don't know what the usenet is, let me explain. Its sort of like a massive database from a the old internet. To get access to this big DB, you need to pay a Usenet provider for access. There are multiple but the one that I used is called Newshosting. The usenet is all a massive jumbled mess of data and files that isn't categorized well. So to help with this you need an Indexer like nzbgeek or DrunkenSlug. Indexers are sort of like a search engine for the usenet that actually lets a human see whats happening. But you don't want to be manually downloading potentially hundreds of files so you need an automated downloader. For me I used SABnzbd but there are many more. Now I can connect this all together in Sonarr and Radarr to find shows and movies for me which are then automatically downloaded!

# Implementation
## Sonarr and Radarr 

1. Make the directory
```bash
sudo mkdir -p /opt/media-stack
cd /opt/media-stack
```

2. Create the docker-compose.yml
```bash
sudo nano docker-compose.yml
```

docker-compose.yml
```yml
services:

  jellyfin:
    image: jellyfin/jellyfin
    container_name: jellyfin
    runtime: nvidia
    environment:
      - NVIDIA_VISIBLE_DEVICES=all
    ports:
      - "8096:8096"
    restart: always
    volumes:
      - jellyfin_config:/config
      - jellyfin_cache:/cache
      - /mnt/media:/media
      - /mnt/media2:/media2

  sonarr:
    image: lscr.io/linuxserver/sonarr:latest
    container_name: sonarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/New_York
    volumes:
      - ./sonarr:/config
      - /mnt/media:/media
      - ./sabnzbd:/sab-config
    ports:
      - "8989:8989"
    restart: unless-stopped

  radarr:
    image: lscr.io/linuxserver/radarr:latest
    container_name: radarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/New_York
    volumes:
      - ./radarr:/config
      - /mnt/media:/media
    ports:
      - "7878:7878"
    restart: unless-stopped
```

3. Launch the stack
```bash
docker compose up -d
```

## NGINX

1. Create config for Radarr
```bash
sudo nano /etc/nginx/sites-available/radarr
```

radarr
```nginx
server {
    listen 80;
    server_name radarr.loganharmondeveloper.com;

    location / {
        proxy_pass http://localhost:7878;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

2. Create config for Sonarr
```bash
sudo nano /etc/nginx/sites-available/sonarr
```

sonarr
```nginx
server {
    listen 80;
    server_name sonarr.loganharmondeveloper.com;

    location / {
        proxy_pass http://localhost:8989;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

3. Enable the sites
```bash
sudo ln -s /etc/nginx/sites-available/radarr /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/sonarr /etc/nginx/sites-enabled/
```

4. Reload NGINX
```bash
sudo nginx -t
sudo systemctl reload nginx
```

## Cloudflare

1. Add entries to the Cloudflared config
```bash
sudo nano /etc/cloudflared/config.yml
```

config.yml
```yml
  - hostname: radarr.loganharmondeveloper.com
    service: http://localhost

  - hostname: sonarr.loganharmondeveloper.com
    service: http://localhost
```

2. Restart the tunnel
```bash
sudo systemctl restart cloudflared
```

3. Add the DNS records
```bash
cloudflared tunnel route dns pterodactyl-tunnel radarr.loganharmondeveloper.com
cloudflared tunnel route dns pterodactyl-tunnel sonarr.loganharmondeveloper.com
```

Now I was able to get into Radarr and Sonarr at my domain now! I set up my admin account with Sonarr and set /media/TV as my Root folder since Sonarr is for tv. Its important to know that when a new storage drive is put in, it has to be mounted first(I would then also set auto mount on boot), then you have to include the media in the volumes for the containers. 
## SABnzbd

1. Edit the media stack
```bash
cd /opt/media-stack
sudo nano docker-compose.yml
```

add to docker-compose.yml
```yml
  sabnzbd:
    image: lscr.io/linuxserver/sabnzbd:latest
    container_name: sabnzbd
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/New_York
    volumes:
      - ./sabnzbd:/config
      - /mnt/media:/media
    ports:
      - "8083:8080"
    restart: unless-stopped
```

2. Start the stack
```bash
docker compose up -d
```

I also went ahead and added sabnzbd to my subdomain and got it working through reverse proxy! It's the same process for NGINX and Cloudflare every time so I am not going to explicitly write it out anymore. Now I got onto the web page and I can see SABnzbd but SAB tries to be very secure and it was giving me a hostname verification failed. So all I need to do to fix this is register my subdomain in the config and SAB will know that my domain is valid.

3. Edit the sabnzbd properties file
```bash
sudo nano /sabnzbd/sabnzbd.ini
```

Look for this line
```ini
host_whitelist =
```
Update it to this
```ini
host_whitelist = sabnzbd.loganharmondeveloper.com,localhost,127.0.0.1,sabnzbd
```

4. Restart the container
```bash
docker restart sabnzbd
```

Now it's up and I can start setting up SABnzbd. SAB asked me for my usenet provider server, the port, and usenet username and password. I didn't have a usenet provider yet so I payed for Newshosting and logged into SAB with the credentials. You can do any usenet provider and it will all work similarly, they have just been running for different amounts of time. For 16 months of access to Newshosting I paid 90 dollars because of a deal. Now I need to connect Sonarr to SAB so that it can find a show, then send it to SAB, and auto-import it into my /media/TV folder on my storage drive. I add SABnzbd to my download clients in Sonarr, then I add NZBGeek and DrunkenSlug into the indexers.

This is most of setup needed to get jellyfin going! I can download my movies from Radarr and shows from Sonarr. They are automatically downloaded and moved onto the jellyfin! There is configuration to do in Sonarr and SAB that I didn't explicitly go over but its not too difficult to figure out yourself or with ai.