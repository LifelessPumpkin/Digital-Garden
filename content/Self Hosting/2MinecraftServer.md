---
title: 2. Minecraft Server
tags:
  - Minecraft
  - Server
  - Ubuntu
  - AMP
  - TailScale
---
# Preamble

So I've always liked playing online games with some friends and I didn't love having to pay 15 dollars a month for a Minecraft server. Since I have my server now I think that this would be a very fun first project to do work on! In the future I am going to improve this and attempt to use containers for the next iteration of my server. I am planning to use kubernetes/docker and host multiple cloud services including a better alternative to AMP. But for now I want to start small and work with AMP to understand some more networking as well.

# Minecraft 

### AMP

This is the game panel that I use for Minecraft. It is a 10 dollar lifetime license for 5 instances including Minecraft and many other games. This was the only purchase I wanted to make.
1. Switch to root
```bash
sudo su -
```

2. Run script
```bash
bash <(wget -qO- getamp.sh)
```

Install runs and I setup my login info. AMP offers a very nice GUI for the Minecraft server and makes things pretty easy. I created an instance on the page and adjusted the memory usage and settings. I started the world and I am able to connect locally from lan IP from this point.  

### TailScale
I didn't want to purchase a domain or ip for a Minecraft server so I did some research and found this mesh VPN service called TailScale. This created my own TailScale IP on my server. Then my friends can install Tailscale from my invite and they are automatically connected to my network.

1. Install TailScale
```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

2. Spin it up
```bash
sudo tailscale up
```

I log in with my email and my server is now a part of my network.
3. Get TailScale IP
```bash
tailscale ip -4
```

Now my friends download TailScale from the link i give them, then they connect to my email and automatically join my network. Now that I have the IP I can ssh and join my Minecraft server from my TailScale IP and server port. TailScale provides peer-2-peer tunneling so that I don't have to change firewall rules or setup port-forwarding! 

> [!NOTE] I have ufw enabled so I still need to allow traffic locally.
