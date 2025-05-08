---
title: 1. Beginning My Personal Cloud
draft: false
tags:
  - Server
  - SelfHosting
  - Ubuntu
  - Webmin
---
# Preamble
My old desktop wasn't running well for a long time and it was bothering me. So I decided to install linux on it and repurpose it into a server. I ran a bunch of diagnostics tests and attempted to find where the computer was crashing. I found this issue with my memory. The memory sticks weren't ordered correctly when they were installed and that caused the issue. I wrote over the windows os and replaced it with Ubuntu 24.04. 
# Operating Sytem Install

My brother had a flashdrive that already had the newest version of Ubuntu so he let me borrow that. I booted to the flashdrive and wrote everything over with Ubuntu. 
# Server Setup

### SSH

```
sudo apt install openssh-server
sudo systemctl enable ssh
```

I had to make sure that port 22 is open on the firewall
### Webmin 

I plan to use the server while in a different city so I want Webmin to monitor the stats of it while I'm gone
1. Download Webmin repo script
```
sudo curl -o setup-repos.sh https://raw.githubusercontent.com/webmin/webmin/master/setup-repos.sh; sudo bash setup-repos.sh
```
2. Run the script
```
sudo bash setup-repos.sh
```
3. Install Webmin
```
sudo apt install --install-recommends webmin -y
```
4. Enable webmin on server start
```
sudo systemctl enable webmin
```

Webmin is now accessible through port 10000 and I can see all my system stats! This is all pretty simple but its a good starting point. I plan to work on a minecraft server next!

![[pikmin_3_berries.webp]]