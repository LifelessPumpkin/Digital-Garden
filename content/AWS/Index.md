---
title: Lightsail Website Creation
---
# Hi! This is all the steps I used to set up my webpage on AWS! 
#### I used Lightsail and set up an Ubuntu 24.04 OS with 2 GB of RAM, 2 vCPUs and a 60 GB SSD. I decided to use a blank Ubuntu Instance instead of the preset LAMP for two reasons. I wanted to set everything up myself from the very beginning. I want full control of the environment.

# Installing Apache 
## Update the package manager
```sh
sudo apt update
```

## Installing Apache2 for the web server
```sh
sudo apt install apache2
```

## List out HTTP Traffic and only allow port 80
```sh
sudo ufw app list
sudo ufw allow in "Apache"
```

## Check to make sure the firewall is disabled
```sh
sudo ufw status
```
#### The Apache server is now up and running! It just shows the default landing page for now but I will change that soon.

# Installing MySQL 
## Install MySQL server
```sh
sudo apt install mysql-server
```

## Run secure installation
```sh
sudo mysql_secure_installation
```

# Installing PHP

## Installing php and some dependencies for apache and mysql
```sh
sudo apt install php libapache2-mod-php php-mysql
```

## Verify installation
```sh
php -v
```

## Change Apache's directory index
```sh
sudo nano /etc/apache2/mods-enabled/dir.conf
```
#### I edited this file by moving the order of index.php so that it takes precedence over index.html

#### I purchased the domain name loganharmondeveloper.com from Route 53

## Creating the virtual host
```sh
sudo mkdir /var/www/loganharmondeveloper
```

## Assign ownership of the directory
```sh
sudo chown -R $USER:$USER /var/www/loganharmondeveloper
```

## Create a new blank configuration file
```sh
sudo nano /etc/apache2/sites-available/loganharmondeveloper.conf
```
## I put this into the new config file

    <VirtualHost *:80>
        ServerName loganharmondeveloper
        ServerAlias www.loganharmondeveloper
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/loganharmondeveloper
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>

## Disable the default site
```sh
sudo a2dissite 000-default
```

## Enable the virtual host
```sh
sudo a2ensite loganharmondeveloper
```

## Run a syntax test
```sh
sudo apache2ctl configtest
```

## Reload Apache 
```sh
sudo systemctl reload apache2
```

## Reload daemons
```sh
sudo systemctl daemon-reload
```

# Now the website is all ready and I will start with the app!

![happy cat :)](https://delavanlakesvet.com/wp-content/uploads/sites/195/2022/03/smiling-cat-for-web.jpg)