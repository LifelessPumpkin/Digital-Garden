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
draft: true
---
# Preamble
So my friends and I have been playing on the server for a little while and its been working great! I found a slight limitation with TailScale. It's free until 5 users. I'm a little sad because I want to do everything on from this point completely open source. Alas I must attempt something different than TailScale. However I think this is also a good thing because I don't want my users to need to download something to get access to the server. So I am going to have to learn more about networking. 
Before I get into that stuff, I first want to start with Pterodactyl. Pterodactyl manages my game servers and is completely open source. Pterodactyl consists of the Panel/Web Interface and the Wings, daemons that run game servers. The wings use docker so I'm going to download that and now I will finally get some experience with containers. Another plus to using Pterodactyl is how customizable it is. I will be able to make more free servers and with a wider span of games on Pterodactyl than I could with AMP.

# Set up

### Docker

1. Add Docker's GPG key
```
sudo apt install -y ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

2. Add Docker's repo
```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

3. Install Docker engine
```
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

4. Test Docker
```
sudo docker run hello-world
```

5. Run Docker without sudo
```
sudo usermod -aG docker $USER
newgrp docker
docker ps
```

### Portainer

1. Install Portainer
```
docker run -d   -p 9443:9443   -p 8000:8000   --name portainer   --restart=always   -v /var/run/docker.sock:/var/run/docker.sock   -v portainer_data:/data   portainer/portainer-ce:latest
```

All I had to do was create a user account and it was all ready!

### Pterodactyl

1. Make proper directories
```
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
```
nano docker-compose.yml
```

docker-compose.yml
```
version: "3.8"
# This section declares the basic config of all of your Containers that are
# declared below as "services"
x-common:
  database:
    &db-environment
    # You don't need to change these because it will not be exposed to the public.
    MYSQL_PASSWORD: &db-password "CHANGE_ME"
    MYSQL_ROOT_PASSWORD: "CHANGE_ME_TOO"
  panel:
    &panel-environment
    #This is the URL that your panel will be on after being reverse proxied.
    # set this to "https://yoursubdomain.yourdomain.yourdomainstld"
    APP_URL: "https://subdomain.domain.tld"
    # A list of valid timezones can be found here:
    # http://php.net/manual/en/timezones.php
    APP_TIMEZONE: "America/New_York"
    APP_SERVICE_AUTHOR: "youremail@gmail.com"
  # Mail is an optional Setup, I have the basic setup if you want to use a gmail
  # account. You will need an App Password as the MAIL_PASSWORD field, not your
  # gmail password. Uncomment the following lines to enable mail.

  #mail:
    #&mail-environment
    #MAIL_FROM: "youremail@gmail.com"
    #MAIL_DRIVER: "smtp"
    #MAIL_HOST: "smtp.gmail.com"
    #MAIL_PORT: "587"
    #MAIL_USERNAME: "youremail@gmail.com"
    #MAIL_PASSWORD: ""
    #MAIL_ENCRYPTION: "true"
services:
  # Wings is the service that hooks into docker and actually creates your game
  # servers,
  wings:
    image: ghcr.io/pterodactyl/wings:latest
    restart: always
    networks:
      - ptero0
    # These are the ports exposed by Wings, I don't recommend changing them.
    ports:
      - "8443:443"
      - "2022:2022"
    tty: true
    environment:
      TZ: "America/New_York"
      # For ease of setup, this is going to use root user.
      WINGS_UID: 0
      WINGS_GID: 0
      WINGS_USERNAME: root
    # This is where docker will bind certain parts of container to your actual
    # host OS. These locations will be used later.
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock" # DO NOT CHANGE
      - "/var/lib/docker/containers:/var/lib/docker/containers" # DO NOT CHANGE
      - "/opt/pterodactyl/wings/config:/etc/pterodactyl" # Feel free to change.
      - "/var/lib/pterodactyl:/var/lib/pterodactyl" # DO NOT CHANGE
      - "/var/log/pterodactyl:/var/log/pterodactyl" # DO NOT CHANGE
      - "/tmp/pterodactyl/:/tmp/pterodactyl/" # Recommended not to change.
  # It's a database. Not much else to explain.
  database:
    image: mariadb:10.5
    restart: always
    command: --default-authentication-plugin=mysql_native_password
    volumes:
      - "/opt/pterodactyl/panel/database:/var/lib/mysql"
    environment:
      <<: *db-environment
      MYSQL_DATABASE: "panel"
      MYSQL_USER: "pterodactyl"
  # It's a CACHE database. Not much else to explain.
  cache:
    image: redis:alpine
    restart: always
  # Now the fun part. Your actual panel.
  panel:
    image: ghcr.io/pterodactyl/panel:latest
    restart: always
    # For NGINX Reverse Proxy, I will be using these ports for simplicity.
    ports:
      - "802:80"
      - "4432:443"
    # Links these containers together in a docker network.
    links:
      - database
      - cache
    # This is where docker will bind certain parts of container to your actual
    # host OS. These don't really matter that much.
    volumes:
      - "/opt/pterodactyl/panel/appvar/:/app/var/"
      - "/opt/pterodactyl/panel/nginx/:/etc/nginx/http.d/"
      - "/opt/pterodactyl/panel/logs/:/app/storage/logs"
    # Sets the config stuff
    environment:
      <<: [*panel-environment]
      # <<: [*mail-environment]
      DB_PASSWORD: *db-password
      APP_ENV: "production"
      APP_ENVIRONMENT_ONLY: "false"
      CACHE_DRIVER: "redis"
      SESSION_DRIVER: "redis"
      QUEUE_DRIVER: "redis"
      REDIS_HOST: "cache"
      DB_HOST: "database"
      DB_PORT: "3306"
# This is Wings' Network. We don't need much depth here, all you need to know, is
# that it allows the passthrough of the ports from Wings.
networks:
  ptero0:
    name: ptero0
    driver: bridge
    ipam:
      config:
        - subnet: "192.55.0.0/16"
    driver_opts:
      com.docker.network.bridge.name: ptero0
```

> [!NOTE] A module I used, 'distutils' was removed from python so I needed to install it

```
sudo apt update
sudo apt install python3-distutils -y
```

3. Run Pterodactyl
```
docker compose up
```

Once it's done spinning up, I hit Ctrl+C to gracefully stop the containers. Then in portainer I start up all containers except the wings.