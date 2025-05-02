---
title: "Tiny HomeLab: Setup pihole with docker"
last_modified_at: 2025-03-08T16:20:02-05:00
categories:
  - Blog
  - Tiny HomeLab
tags:
  - Tiny HomeLab
  - Raspberry pi project
  - pihole
  - docker compose
---

If you already have a raspberry pi (Or any home server) with linux and docker installed, setting up pihole is quite simple.
I prefer using docker-compose files to manage the services on my raspberry-pi instead of using docker run commands, since the configuration is at one place and managing the service is one "docker-compose up/down"command away.

### To Setup
- Create an empty folder:
    ```
    mkdir pihole
    ```
- Crete a file named **docker-compose.yaml** file in the directory **pihole**.
    ```
    touch  pihole/docker-compose.yaml
    ``` 
- Here is what the **docker-compose.yaml** should look like
    ```dockerfile
    version: "3"
    # More info at https://github.com/pi-hole/docker-pi-hole/ and https://docs.pi-hole.net/
    services:
    pihole:
        container_name: pihole
        image: pihole/pihole:latest
        # For DHCP it is recommended to remove these ports and instead add: network_mode: "host"
        ports:
        - "53:53/tcp"
        - "53:53/udp"
        - "67:67/udp" # Only required if you are using Pi-hole as your DHCP server
        - "8080:80/tcp"
        environment:
        TZ: 'Asia/Calcutta'
        WEBPASSWORD: 'password' # This is the password for the web UI.
        # Volumes store your data between container upgrades
        volumes:
        - './etc-pihole:/etc/pihole'
        - './etc-dnsmasq.d:/etc/dnsmasq.d'    
        #   https://github.com/pi-hole/docker-pi-hole#note-on-capabilities
        cap_add:
        - NET_ADMIN # Recommended but not required (DHCP needs NET_ADMIN)      
        restart: always

    ```
- Start the docker with following command, this will download the images and start the container
    ```
    docker compose up -d
    ``` 
- Once the container is ready, web UI is available at
    ```
    http://<your-pihole-ip>/admin
    ```
- Once configured you would need to change the DNS IP of your home router to the pihole IP.

### To stop the service
- use following command
    ```
    docker compose down
    ```

### To Update
- Update using following command
    ```
    docker compose pull && docker compose up -d
    ```
- Prune the unused old images
    ```
    docker image prune
    ```
