---
title: "Tiny HomeLab: Setup Kanboard"
last_modified_at: 2025-03-08T16:20:02-05:00
categories:
  - Blog
  - Tiny HomeLab
tags:
  - Tiny HomeLab
  - Raspberry pi project
  - kanboard
  - docker compose
---

[Kanboard](https://kanboard.org/) is a free and open source Kanban project management software. It's lightweight, clean an great for managing personal and small projects.

![Kanboard Project page](https://kanboard.org/assets/img/board.png "Kanboard")

### To Setup
- Create an empty folder:
    ```
    mkdir kanboard
    ```
- Crete a file named **docker-compose.yaml** file in the directory **kanboard**.
    ```
    touch  kanboard/docker-compose.yaml
    ``` 
- Here is what the **docker-compose.yaml** should look like
    ```dockerfile
  version: '2'
  services:
    kanboard:
      container_name: kanboard
      image: kanboard/kanboard:latest
      ports:
        - "4000:443"
      restart: always
      environment:
        - PLUGIN_INSTALLER=true
      volumes:
        - ./kanboard_data:/var/www/app/data
        - ./kanboard_plugins:/var/www/app/plugins
        - ./kanboard_ssl:/etc/nginx/ssl
    ```
- Create following folders inside **kanboard** directory:
    ```
    mkdir kanboard_data
    mkdir kanboard_plugins
    mkdir kanboard_ssl
    ```
- Start the docker with following command, this will download the images and start the container
    ```
    docker compose up -d
    ``` 
- Once the container is ready, web UI is available at
    ```
    http://<your-kanboard-server-ip>/4000
    ```

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