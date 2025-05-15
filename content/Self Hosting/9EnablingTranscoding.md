---
title: 9. Enabling Transcoding
tags:
  - Transcoding
  - GPU
  - Jellyfin
  - Media
  - Docker
  - NVIDIA
  - Containers
draft: false
---
# Preamble

Since I have my GPU in my server, I thought that it would be a good idea to use it for video transcoding. It's not gonna be necessary that often but I figure theres no harm.

# Implementation

## NVIDIA
1. Install NVIDIA drivers
```bash
sudo apt install nvidia-driver-525
```

2. Reboot
```bash
sudo reboot
```

3. Test it
```bash
nvidia-smi
```

You should see output about your GPU.

3. Install Docker Toolkit
```bash
curl -s -L https://nvidia.github.io/nvidia-docker/ubuntu22.04/nvidia-docker.list | \
  sudo tee /etc/apt/sources.list.d/nvidia-docker.list
sudo apt update
sudo apt install -y nvidia-container-toolkit
sudo systemctl restart docker
```

4. Update jellyfin in the media-stack docker-compose
```bash
sudo nano docker-compose.yml
```

```yml
  jellyfin:
    image: jellyfin/jellyfin
    container_name: jellyfin
    runtime: nvidia
    environment:
      - NVIDIA_VISIBLE_DEVICES=all
    ports:
      - 8096:8096
    restart: always
    volumes:
      - jellyfin_config:/config
      - jellyfin_cache:/cache
      - /mnt/media:/media
```

5. Edit Docker Daemon Config
```bash
sudo nano /etc/docker/daemon.json
```

paste in daemon.json
```json
{
  "default-runtime": "nvidia",
  "runtimes": {
    "nvidia": {
      "path": "nvidia-container-runtime",
      "runtimeArgs": []
    }
  }
}
```

6. Restart Docker
```bash
sudo systemctl restart docker
```

7. Restart the container
```bash
docker compose down
docker compose up -d
```

At this point I go into jellyfin UI and enable transcoding with Nvidia NVENC!