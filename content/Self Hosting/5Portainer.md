---
title: 5. Portainer
tags:
  - Portainer
  - Docker
  - Containers
  - NGINX
  - Cloudflare
  - Tunnels
  - Server
---
# Preamble

The next subdomain I'm working on is Portainer. I want to have the nice GUI that it provides for my containers since I am going to have many. Maybe this could help for when I eventually use kubernetes.

# Implementation
## Cloudflare Tunnel

1. Edit the Tunnel Config
```bash
sudo nano /etc/cloudflared/config.yml
```

config.yml
```yml
tunnel: pterodactyl-tunnel
credentials-file: /home/logan/.cloudflared/pterodactyl-tunnel.json

ingress:
  - hostname: panel.loganharmondeveloper.com
    service: http://localhost

  - hostname: portainer.loganharmondeveloper.com
    service: http://localhost

  - service: http_status:404
```

2. Create DNS route for Portainer
```bash
cloudflared tunnel route dns pterodactyl-tunnel portainer.loganharmondeveloper.com
```

3. Make Portainer run on port 9000
```bash
docker stop portainer
docker rm portainer
docker run -d \
  --name=portainer \
  -p 9000:9000 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  --restart=always \
  portainer/portainer-ce
```

## NGINX

Since Portainer already uses https automatically it can cause some problems because Cloudflare is also trying to provide SSL. So I am going to use NGINX in the loop for Portainer in http so that Cloudflare will provide the SSL.

1. Create a new config file
```bash
sudo nano /etc/nginx/sites-available/portainer
```

portainer
```nginx
server {
    listen 80;
    server_name portainer.loganharmondeveloper.com;
    
    location / {
        proxy_pass http://localhost:9000;
        proxy_ssl_verify off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

2. Symbolic link and reload
```bash
sudo ln -s /etc/nginx/sites-available/portainer /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

Now I'm able to go to https://portainer.loganharmondeveloper.com and I can see the login!

![[pikmin-4-review-gameshub.webp]]