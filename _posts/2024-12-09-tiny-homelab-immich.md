---
title: "Tiny HomeLab: Setup immich with docker"
last_modified_at: 2025-03-08T16:20:02-05:00
categories:
  - Blog
  - Tiny HomeLab
tags:
  - Tiny HomeLab
  - Raspberry pi project
  - immich
  - docker compose
---

[Immich](https://immich.app/) is an excellent self hosted replacement for the popular cloud based image hosting software. It includes features like, Android and IOS apps, automatic sync, multi user support, AI face recognition among others.

### To Setup
- Create an empty folder:
    ```
    mkdir immich
    ```
- Crete a file named **docker-compose.yaml** file in the directory **pihole**.
    ```
    touch  immich/docker-compose.yaml
    ``` 
- Here is what the **docker-compose.yaml** should look like
    ```dockerfile
  services:
  immich-server:
    container_name: immich_server
    image: ghcr.io/immich-app/immich-server:${IMMICH_VERSION:-release}
    # extends:
    #   file: hwaccel.transcoding.yml
    #   service: cpu # set to one of [nvenc, quicksync, rkmpp, vaapi, vaapi-wsl] for accelerated transcoding
    volumes:
      # Do not edit the next line. If you want to change the media storage location on your system, edit the value of UPLOAD_LOCATION in the .env file
      - ${UPLOAD_LOCATION}:/usr/src/app/upload
      - /etc/localtime:/etc/localtime:ro
    env_file:
      - .env
    ports:
      - 2283:2283
    depends_on:
      - redis
      - database
    restart: always
    healthcheck:
      disable: false

  immich-machine-learning:
    container_name: immich_machine_learning
    # For hardware acceleration, add one of -[armnn, cuda, openvino] to the image tag.
    # Example tag: ${IMMICH_VERSION:-release}-cuda
    image: ghcr.io/immich-app/immich-machine-learning:${IMMICH_VERSION:-release}
    # extends: # uncomment this section for hardware acceleration - see https://immich.app/docs/features/ml-hardware-acceleration
    #   file: hwaccel.ml.yml
    #   service: cpu # set to one of [armnn, cuda, openvino, openvino-wsl] for accelerated inference - use the `-wsl` version for WSL2 where applicable
    volumes:
      - model-cache:/cache
    env_file:
      - .env
    dns: # optional
      - 8.8.8.8
    restart: always

  redis:
    container_name: immich_redis
    image: docker.io/redis:6.2-alpine@sha256:e3b17ba9479deec4b7d1eeec1548a253acc5374d68d3b27937fcfe4df8d18c7e
    healthcheck:
      test: redis-cli ping || exit 1
    restart: always

  database:
    container_name: immich_postgres
    image: docker.io/tensorchord/pgvecto-rs:pg14-v0.2.0@sha256:90724186f0a3517cf6914295b5ab410db9ce23190a2d9d0b9dd6463e3fa298f0
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_USER: ${DB_USERNAME}
      POSTGRES_DB: ${DB_DATABASE_NAME}
      POSTGRES_INITDB_ARGS: '--data-checksums'
    volumes:
      # Do not edit the next line. If you want to change the database storage location on your system, edit the value of DB_DATA_LOCATION in the .env file
      - ${DB_DATA_LOCATION}:/var/lib/postgresql/data
    healthcheck:
      test: pg_isready --dbname='${DB_DATABASE_NAME}' --username='${DB_USERNAME}' || exit 1; Chksum="$$(psql --dbname='${DB_DATABASE_NAME}' --username='${DB_USERNAME}' --tuples-only --no-align --command='SELECT COALESCE(SUM(checksum_failures), 0) FROM pg_stat_database')"; echo "checksum failure count is $$Chksum"; [ "$$Chksum" = '0' ] || exit 1
      interval: 5m
      start_interval: 30s
      start_period: 5m
    command: ["postgres", "-c" ,"shared_preload_libraries=vectors.so", "-c", 'search_path="$$user", public, vectors', "-c", "logging_collector=on", "-c", "max_wal_size=2GB", "-c", "shared_buffers=512MB", "-c", "wal_compression=on"]
    restart: always
  
  # Restore https://immich.app/docs/administration/backup-and-restore/ 

    volumes:
    model-cache:
    ```
- create a file name .env in the "immich" directory and add following content, update the values as needed.
    ```
    # You can find documentation for all the supported env variables at https://immich.app/docs/install/environment-variables

    # The location where your uploaded files are stored
    UPLOAD_LOCATION=./immich_data/library
    # The location where your database files are stored
    DB_DATA_LOCATION=/home/atul/docker-compose-services/immich/postgres

    # To set a timezone, uncomment the next line and change Etc/UTC to a TZ identifier from this list: https://en.wikipedia.org/wiki/List_of_tz_database_time_zones#List
    # TZ=Etc/UTC

    # The Immich version to use. You can pin this to a specific version like "v1.71.0"
    IMMICH_VERSION=release

    # Connection secret for postgres. You should change it to a random password
    DB_PASSWORD=<password>

    # The values below this line do not need to be changed
    ###################################################################################
    DB_USERNAME=postgres
    DB_DATABASE_NAME=immich

    ```
- Create a directory named "immich_data", this directory will be mounted to "immich_server" container and will store your images inside immich directory.
    ```
    mkdir immich_data
    ```
- Start the docker with following command, this will download the images and start the container
    ```
    docker compose up -d
    ``` 
- Once the container is ready, web UI is available at
    ```
    http://<your-immich-server-ip>:2883
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
- More details at [https://immich.app/docs/install/upgrading/](https://immich.app/docs/install/upgrading/)

### Backup and restore
- [https://immich.app/docs/administration/backup-and-restore/](https://immich.app/docs/administration/backup-and-restore/)
