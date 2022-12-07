---
layout: post
title: 'Homelab'
---

Documentation for my homelab servers.

Current servers:
1. Ubuntu server
2. Raspberry Pi Zero W server
3. Raspberry Pi 3B+ server

#### 1.0 Ubuntu Server

I converted an old laptop from 2014 with dead pixels in the screen into a server. Since it runs headless, the dead pixels aren't a problem.

Current services I run on it:
1. Docker
2. Uptime Kuma
3. Combustion
4. Filebrowser
5. Jellyfin
6. Jupyterlab
7. Kavita
8. Pi-Hole
9. Tailscale
10. Synthing
11. rsync

{% image="projects/homelab/server.jpg" %}

#### 1.1 Docker
- All services are virtualized using Docker.
- Docker-Compose makes it easy to spin up, modify, and spin down images.

```
sudo apt install docker && docker-compose -y
```

#### 1.2 Uptime Kuma
- Monitor services to see if they go down.

```
uptime-kuma:
    image: louislam/uptime-kuma:1
    container_name: uptime-kuma
    volumes:
        - [local folder path]:/app/data
    ports:
        - 80:[port to access]  # <Host Port>:<Container Port>
    restart: always

```

#### 1.3 Combustion
- Download and seed Linux ISO's.

```
transmission:
    image: lscr.io/linuxserver/transmission:latest
    container_name: transmission
    environment:
        - PUID=1000
        - PGID=1000
        - TZ=Asia/Manila
        - TRANSMISSION_WEB_HOME=/combustion-release/ #optional
    volumes:
        - [local folder path]/config:/config
        - [local folder path]/Torrents:/downloads
        - [local folder path]/watch:/watch
    ports:
        - 9091:[port to access]
        - 51413:[port to access]
        - 51413:[port to access]/udp
    restart: unless-stopped
```

#### 1.4 Filebrowser
- Browse files in a web browser.

```
filebrowser:
    image: hurlenko/filebrowser
    user: "1000:1000"
    ports:
        - 8080:[port to access]
    volumes:
        - [local folder path]:/data
        - [local folder path]/config:/config
    environment:
        - FB_BASEURL=/filebrowser
    restart: always
```

#### 2.0 Pi-Zero
Power directly by my router, so both turn on and off at the same time.

Current services I run on it:
1. Pi-Hole
2. Tailscale

#### 3.0 Pi 3B+
Serves as a back-up NAS.

Current services I run on it:
1. Docker
2. Syncthing
3. rsync
4. Tailscale