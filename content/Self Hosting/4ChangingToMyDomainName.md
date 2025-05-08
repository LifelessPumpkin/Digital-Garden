---
title: 4. Changing to my Domain Name
draft: false
tags:
  - Server
  - Portainer
  - Docker
  - ReverseProxy
  - Tunnels
  - Route53
  - AWS
  - NGINX
  - Pterodactyl
  - DNS
  - Cloudflare
---

# Preamble

So I didn't want to be accessing my server's information through the lan/Tailscale ip for its lifecycle. I decided Cloudflare would be great to use. Mainly because I didn't want to buy an SSL cert. I would set up the subdomains from my personal website, loganharmondeveloper.com on Cloudflare. Currently it's hosted on route 53 so I needed to change the hosted zones from AWS' name-servers to Cloudflare's name-servers. It was pretty simple I just went into route 53 and edited them.

The only problem I have with Cloudflare is the latency it adds. For most things its not really a problem like webmin, pterodactyl panel, Jellyfin, etc. But for playing games(my Minecraft server) it would add some latency that I don't really want. So I decided to go with a split approach. The normal web stuff will run through Cloudflare and the pterodactyl wings(game servers) will run through Tailscale. I can't get an IP from my ISP because Frontier only does it for business accounts(and it's like 500 dollars lol) so I'm going to stick with Tailscale for now. The only limitation I have with that is that it can only host 6 people, so I will try to find some alternative later. My next step is installing NGINX for the reverse proxy layer.

### NGINX

1. Install NGINX
```bash
sudo apt update
sudo apt install nginx -y
```

2. Check to make sure it's running
```bash
sudo systemctl start nginx
```

>[!WARNING] Be careful for port conflicts!

### Cloudflared
1. Download the repo
```bash
curl -fsSL https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb -o cloudflared.deb
```

2. Install it
```bash
sudo dpkg -i cloudflared.deb
```

3. Login to Cloudflare
```bash
cloudflared login
```

At this point I login to Cloudflare in my browser and select my domain. Once I login it automatically saves a cert.

4. Create the tunnel
```bash
cloudflared tunnel create pterodactyl-tunnel
```

5. Create the Cloudflared directory
```bash
sudo mkdir -p /etc/cloudflared
```
5. Create the config file
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

  - service: http_status:404
```

6. Change the DNS record in Cloudflare
```bash
cloudflared tunnel route dns pterodactyl-tunnel panel.loganharmondeveloper.com
```

7. Test the tunnel
```bash
cloudflared tunnel run pterodactyl-tunnel
```

8. Install Cloudflared as a service
```bash
sudo cloudflared service install
```

9. Check that the service is running and if it's not then start it
```bash
sudo systemctl status cloudflared
```

10. Enable it on boot
```bash
sudo systemctl enable cloudflared
```

At this point I can go to panel.loganharmondeveloper.com and it loads nginx so I need to change the config.

11. Edit the NGINX config
```bash
sudo nano /etc/nginx/sites-available/pterodactyl
```

pterodactyl
```nginx
server {
    listen 80;
    server_name panel.loganharmondeveloper.com;

    location / {
        proxy_pass http://127.0.0.1:82;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

12. Symbolic Link
```bash
sudo ln -s /etc/nginx/sites-available/pterodactyl /etc/nginx/sites-enabled/
```

13. Test it
```bash
sudo nginx -t
```

14. Remove the default site
```bash
sudo rm /etc/nginx/sites-enabled/default
sudo systemctl reload nginx
```

Now I can go to https://panel.loganharmondeveloper.com and I go to my Pterodactyl panel! Since I have NGINX listening on port 80, Cloudflare handles all the HTTPS to the user. Im going to get the rest of my services running on my subdomain next!

![[bulborb_eating.png]]