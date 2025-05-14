---
title: 6. Webmin
tags:
  - Webmin
  - Cloudflare
  - DNS
  - Server
  - ReverseProxy
  - NGINX
---
# Preamble

I am going to add webmin to a subdomain so I can see my system stats.

# Implementation

## Update Cloudflare
1. Edit Cloudflared tunnel config
```bash
sudo nano /etc/cloudflared/config.yml
```

config.yml
```yaml
tunnel: pterodactyl-tunnel
credentials-file: /home/logan/.cloudflared/pterodactyl-tunnel.json  

ingress:
  - hostname: panel.loganharmondeveloper.com
    service: http://localhost

  - hostname: portainer.loganharmondeveloper.com
    service: http://localhost

  - hostname: webmin.loganharmondeveloper.com
    service: http://localhost

  - service: http_status:404
```

2. Restart Cloudflare
```bash
sudo systemctl restart cloudflared
```

3. Create DNS entry
```bash
cloudflared tunnel route dns pterodactyl-tunnel webmin.loganharmondeveloper.com
```
## Update NGINX

1. Edit NGINX config
```bash
sudo nano /etc/nginx/sites-available/webmin
```

```nginx
server {
    listen 80;
    server_name webmin.loganharmondeveloper.com;

    location / {
        proxy_pass https://localhost:10000;
        proxy_ssl_verify off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

3. Symbolic link and reload
```bash
sudo ln -s /etc/nginx/sites-available/webmin /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

Now I tried getting to webmin.loganharmondeveloper.com from here and I could see the login page but when I attempted to login I was denied. I was able to make curls to the port from the server and webmin was confirmed to be running. The problem was in the configuration file of webmin.

## Webmin Fix

1. Edit the config
```bash
sudo nano /etc/webmin/miniserv.conf
```

2. Add these lines to miniserv.conf
```conf
port=10000
listen=
referers=webmin.loganharmondeveloper.com
```

I waited a bit and refreshed and I was able to get to my webmin panel!