---
title: 2. Pterodactyl
tags:
  - Server
  - Minecraft
  - Pterodactyl
  - Wing
  - Docker
  - Portainer
  - ReverseProxy
draft: false
---
# Preamble
So my friends and I have been playing on the server for a little while and its been working great! I found a slight limitation with TailScale. It's free until 5 users. I'm a little sad because I want to do everything on from this point completely open source. Alas I must attempt something different than TailScale. However I think this is also a good thing because I don't want my users to need to download something to get access to the server. So I am going to have to learn more about networking. 
Before I get into that stuff, I first want to start with Pterodactyl. Pterodactyl manages my game servers and is completely open source. Pterodactyl consists of the Panel/Web Interface and the Wings, daemons that run game servers. The wings use docker so I'm going to download that and now I will finally get some experience with containers. Another plus to using Pterodactyl is how customizable it is. I will be able to make more free servers and with a wider span of games on Pterodactyl than I could with AMP.

# Set up

### Docker

1. Add Docker's GPG key
```bash
sudo apt install -y ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

2. Add Docker's repo
```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

3. Install Docker engine
```bash
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

4. Test Docker
```bash
sudo docker run hello-world
```

5. Run Docker without sudo
```bash
sudo usermod -aG docker $USER
newgrp docker
docker ps
```

### Portainer

1. Install Portainer
```bash
docker run -d   -p 9443:9443   -p 8000:8000   --name portainer   --restart=always   -v /var/run/docker.sock:/var/run/docker.sock   -v portainer_data:/data   portainer/portainer-ce:latest
```

All I had to do was create a user account and it was all ready!

### Pterodactyl

1. Make proper directories
```bash
mkdir /opt/pterodactyl
cd /opt/pterodactyl/
mkdir wings
mkdir wings/config
mkdir panel
mkdir panel/appvar
mkdir panel/nginx
mkdir panel/logs
```

2. Create Dockerfile
```bash
nano docker-compose.yml
```

docker-compose.yml
```yml
version: '3.8'

x-common:
  database: &db-environment
    MYSQL_PASSWORD: &db-password "secret :)"
    MYSQL_ROOT_PASSWORD: "secret :)"
  panel: &panel-environment
    APP_URL: "http://loganharmondeveloper.com"
    APP_TIMEZONE: "UTC"
    APP_SERVICE_AUTHOR: "logan3harmon@gmail.com"
    # LE_EMAIL: ""

services:
  wings:
    image: ghcr.io/pterodactyl/wings:latest
    restart: always
    networks:
      - wings0
    ports:
      - "8081:8081"
      - "2022:2022"
    tty: true
    environment:
      TZ: "UTC"
      WINGS_UID: 988
      WINGS_GID: 988
      WINGS_USERNAME: pterodactyl
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
      - "/var/lib/docker/containers/:/var/lib/docker/containers/"
      - "/etc/pterodactyl/:/etc/pterodactyl/"
      - "/var/lib/pterodactyl/:/var/lib/pterodactyl/"
      - "/var/log/pterodactyl/:/var/log/pterodactyl/"
      - "/tmp/pterodactyl/:/tmp/pterodactyl/"
      - "/etc/ssl/certs:/etc/ssl/certs:ro"

  database:
    image: mariadb:10.5
    restart: always
    command: --default-authentication-plugin=mysql_native_password
    volumes:
      - "/srv/pterodactyl/database:/var/lib/mysql"
    environment:
      <<: *db-environment
      MYSQL_DATABASE: "panel"
      MYSQL_USER: "pterodactyl"

  cache:
    image: redis:alpine
    restart: always

  panel:
    image: ghcr.io/pterodactyl/panel:latest
    restart: always
    ports:
      - "82:80"
      - "4432:443"
    links:
      - database
      - cache
    volumes:
      - "/srv/pterodactyl/var/:/app/var/"
      - "/srv/pterodactyl/nginx/:/etc/nginx/http.d/"
      # - "/srv/pterodactyl/certs/:/etc/letsencrypt/"
      - "/srv/pterodactyl/logs/:/app/storage/logs"
    environment:
      <<: [*panel-environment]
      DB_PASSWORD: *db-password
      APP_ENV: "production"
      APP_ENVIRONMENT_ONLY: "false"
      CACHE_DRIVER: "redis"
      SESSION_DRIVER: "redis"
      QUEUE_DRIVER: "redis"
      REDIS_HOST: "cache"
      DB_HOST: "database"
      DB_PORT: "3306"

networks:
  wings0:
    name: wings0
    driver: bridge
    ipam:
      config:
        - subnet: "172.21.0.0/16"
    driver_opts:
      com.docker.network.bridge.name: wings0
```

I created this docker-compose file from the [Pterodactyl Github ](https://github.com/pterodactyl) repo examples in the panel and wings repo. Furthermore I was missing the distutils library so I installed that 

```bash
sudo apt update
sudo apt install python3-distutils -y
```

3. Run Pterodactyl
```bash
docker compose up
```

Once it's done spinning up, it will give errors because wings doesn't have the config file it wants. So I hit Ctrl+C to gracefully stop all the containers. Then in portainer I start up all containers except the wings. I can go to the pterodactyl panel now at port 82!

From this point on, I completed 4. Changing to my Domain Name and I'm adding to this from that point.