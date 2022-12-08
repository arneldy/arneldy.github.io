---
layout: post
title: 'Homelab'
---

Documentation for my homelab servers.

Current servers:
1. Ubuntu server
2. Raspberry Pi Zero W server
3. Raspberry Pi 3B+ server

![Server](https://raw.githubusercontent.com/arneldy/arneldy.github.io/gh-pages/assets/img/projects/homelab/server.jpg)

#### 1.0 Ubuntu Server

I converted an old laptop I bought back in 2014 into a server. I decommissioned it from work after 6 or so years because of the dead pixels on the screen. Since a server can run headless, it's not a big problem.

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
12. TLP

![Server](https://raw.githubusercontent.com/arneldy/arneldy.github.io/gh-pages/assets/img/projects/homelab/thumbnail.jpg)

#### 1.1 Docker
- All services are virtualized using Docker.
- Docker-Compose makes it easy to spin up, modify, and spin down images.

1. Install Docker and Docker-Compose.
```s
sudo apt install docker && apt install docker-compose -y
```

2. Make a docker-compose.yml (see configuration files per service below for examples).
3. Navigate to folder containing docker-compose.yml.
4. Spin up docker volumes.
```s
sudo docker-compose up -d
```

5. Update images as needed.
```s
sudo docker-compose up --build --force-recreate -d
```

6. Spin down docker volumes.
```s
sudo docker-compose down
```

#### 1.2 Uptime Kuma
- Monitor services to see if they go down.

```
uptime-kuma:
    image: louislam/uptime-kuma:1
    container_name: uptime-kuma
    volumes:
        - [local folder path]/uptime_kuma/uptime-kuma-data:/app/data
    ports:
        - <Host Port>:80  # <Host Port>:<Container Port>
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
        - [local folder path]/transmission/config:/config
        - [local folder path]/Torrents:/downloads
        - [local folder path]/transmission/watch:/watch
    ports:
        - <Host Port>:9091
        - <Host Port>:51413
        - <Host Port>:51413/udp
    restart: unless-stopped
```

#### 1.4 Filebrowser
- Browse local files in a web browser.

```
filebrowser:
    image: hurlenko/filebrowser
    user: "1000:1000"
    ports:
        - <Host Port>:8080
    volumes:
        - [local folder path]:/data
        - [local folder path]/filebrowser/config:/config
    environment:
        - FB_BASEURL=/filebrowser
    restart: always
```

#### 1.5 Jellyfin
- Open source Netflix-like service.

```
jellyfin:
    image: jellyfin/jellyfin
    container_name: jellyfin
    user: 1000:1000
    network_mode: "host"
    volumes:
        - [local folder path]/config:/config
        - [local folder path]/cache:/cache
        - [local folder path]:/media
        - [local folder path]:/media2
    restart: "unless-stopped"
```

#### 1.6 Jupyterlab
- Can code and run programs remotely.

```
  jupyter:
      image: jupyter/scipy-notebook:latest
      container_name: jupyter
      ports:
        - <Host Port>:8888
      volumes:
        - [local folder path]:/home/jovyan/work
      environment:
        JUPYTER_ENABLE_LAB: "yes"
        JUPYTER_TOKEN: "[password]"
```

#### 1.7 Kavita
- Library service.

```
kavita:
    image: kizaing/kavita:latest    # Change latest to nightly for latest develop builds (can't go back to stable)
        container_name: kavita
        volumes:
        - [local folder path]/books:/books
        - [local folder path]/kavita/config:/kavita/config
      environment:
        - TZ=Asia/Manila
      ports:
        - "<Host Port>:5000" # Change the public port (the first 5000) if you have conflicts with other services
      restart: unless-stopped
```

#### 1.8 Pi-Hole
- Blocks ads and malware on the network level.

```
pihole:
    container_name: pihole
    image: pihole/pihole:latest
    # For DHCP it is recommended to remove these ports and instead add: network_mode: "host"
    ports:
      - "<Host Port>:53/tcp"
      - "<Host Port>:53/udp"
      - "<Host Port>:80/tcp"
    environment:
      TZ: 'Asia/Manila'
      # WEBPASSWORD: 'set a secure password here or it will be random'
    # Volumes store your data between container upgrades
    volumes:
      - '[local folder path]/pihole/etc-pihole:/etc/pihole'
      - '[local folder path]/pihole/etc-dnsmasq.d:/etc/dnsmasq.d'
    #   https://github.com/pi-hole/docker-pi-hole#note-on-capabilities
    restart: unless-stopped
```

#### 1.9 Tailscale
- Easy internal VPN.

```
tailscale:
    privileged: true
    hostname: tailscale                                          # This will become the tailscale device name
    network_mode: "host"
    container_name: tailscale
    image: tailscale/tailscale:latest
    volumes:
        - "[local folder path]/tailscale/var_lib:/var/lib"        # State data will be stored in this directory
        - "/dev/net/tun:/dev/net/tun"                      # Required for tailscale to work
    cap_add:                                               # Required for tailscale to work
        - net_admin
        - sys_module
    command: tailscaled
    restart: unless-stopped

```

#### 1.10 Synthing
- Synchronize and mirror files.

```
syncthing:
    image: lscr.io/linuxserver/syncthing:latest
    container_name: syncthing
    hostname: ubuntuserver #optional
    environment:
        - PUID=1000
        - PGID=1000
        - TZ=Asia/Manila
    volumes:
        - [local folder path]/syncthing/config:/config
        - [local folder path]:/sda
        - [local folder path]:/sdb
    ports:
        - <Host Port>:8384
        - <Host Port>:22000/tcp
        - <Host Port>:22000/udp
        - <Host Port>:21027/udp
    restart: unless-stopped
```

#### 1.11 rsync
- CLI file transfer.
- crontab -e bash script for automated NAS back-up.

#### 1.12 TLP
- Battery management, good for laptops that are always plugged in.
- A plus for laptops turned into servers is that the battery is effectively a UPS.
- Can set to charge max 60%.

1. Install TLP.

```s
sudo apt install tlp
```

2. Make configuration file.

```s
mkdir -p /etc/tlp.d
cp /etc/tlp.conf /etc/tlp.d/01-myconfig.conf
```

3. Configure.

```
#Set to 0 to disable, 1 to enable TLP.
#Default: 1
#TLP_ENABLE=1
```

4. Start service.

```s
sudo tlp start
```

5. Monitor service.

```s
sudo tlp-stat -s -c -b
```

#### 2.0 Pi-Zero
Powered directly by my router, so both turn on and off at the same time.

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