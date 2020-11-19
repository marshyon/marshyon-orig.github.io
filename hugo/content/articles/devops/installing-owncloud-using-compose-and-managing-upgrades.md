---
title: "Installing OwnCloud using Compose on a server running docker in a home network"
date: 2020-11-19T14:46:58+01:00
draft: false
---


# installing OwnCloud 

create and change directory to a new directory in which to manage this compose application with :

```
mkdir ~/owncloud-docker-server
cd ~/owncloud-docker-server
```

create a file called `.env` with the following :

```
OWNCLOUD_VERSION=10.5
OWNCLOUD_DOMAIN=<hostname or ip address of your docker server>
ADMIN_USERNAME=<admin user>
ADMIN_PASSWORD=<admin password>
HTTP_PORT=8080
```

being sure to edit in the ip address or hostname of the server that is running your docker instance on your home network, an admin user and an appropriate password. I like to use a password generator as is found in tools like 'LastPass', 'BitWarden' and 'keepass' to create random long and strong passwords but that is for you to decide which or what to use for this.

next create a file called `docker-compose.yaml` containing the following :

```
version: '2.1'

services:
  owncloud:
    image: owncloud/server:${OWNCLOUD_VERSION}
    restart: always
    ports:
      - ${HTTP_PORT}:8080
    depends_on:
      - db
      - redis
    environment:
      - OWNCLOUD_DOMAIN=${OWNCLOUD_DOMAIN}
      - OWNCLOUD_DB_TYPE=mysql
      - OWNCLOUD_DB_NAME=owncloud
      - OWNCLOUD_DB_USERNAME=owncloud
      - OWNCLOUD_DB_PASSWORD=owncloud
      - OWNCLOUD_DB_HOST=db
      - OWNCLOUD_ADMIN_USERNAME=${ADMIN_USERNAME}
      - OWNCLOUD_ADMIN_PASSWORD=${ADMIN_PASSWORD}
      - OWNCLOUD_MYSQL_UTF8MB4=true
      - OWNCLOUD_REDIS_ENABLED=true
      - OWNCLOUD_REDIS_HOST=redis
    healthcheck:
      test: ["CMD", "/usr/bin/healthcheck"]
      interval: 30s
      timeout: 10s
      retries: 5
    volumes:
      - ./files:/mnt/data

  db:
    image: webhippie/mariadb:latest
    restart: always
    environment:
      - MARIADB_ROOT_PASSWORD=owncloud
      - MARIADB_USERNAME=owncloud
      - MARIADB_PASSWORD=owncloud
      - MARIADB_DATABASE=owncloud
      - MARIADB_MAX_ALLOWED_PACKET=128M
      - MARIADB_INNODB_LOG_FILE_SIZE=64M
    healthcheck:
      test: ["CMD", "/usr/bin/healthcheck"]
      interval: 30s
      timeout: 10s
      retries: 5
    volumes:
      - ./mysql:/var/lib/mysql
      - ./backup:/var/lib/backup

  redis:
    image: webhippie/redis:latest
    restart: always
    environment:
      - REDIS_DATABASES=1
    healthcheck:
      test: ["CMD", "/usr/bin/healthcheck"]
      interval: 30s
      timeout: 10s
      retries: 5
    volumes:
      - ./redis:/var/lib/redis
```

now finally invoke docker-compose to initially download the docker images used in this application and start each up as running containers with :

```
docker-compose up -d
```

This may take from a few seconds to minutes or more depending on your internet connectivity as each of the 3 images needed will be downloaded from Docker Hub if you do not already have them downloaded in previous installations.

if something like the ( truncated ) output is seen ( below ) there is a good chance that Owncloud is running :

```
...
5e62f6d7a9e0: Pull complete
f428e932ff4b: Pull complete
Digest: sha256:ef46405c1a47ea63f5eaea4adb2e96a8c7bf85ccfd35fd1097df71381f8aeee6
Status: Downloaded newer image for owncloud/server:10.5
Creating owncloud-docker-server_db_1    ... done
Creating owncloud-docker-server_redis_1 ... done
Creating owncloud-docker-server_owncloud_1 ... done
```

confirm that the 3 containers are indeed running with `docker ps` :

```
$ docker ps
CONTAINER ID        IMAGE                      COMMAND                  CREATED              STATUS                        PORTS                    NAMES
2afb71f575d3        owncloud/server:10.5       "/usr/bin/entrypoint…"   About a minute ago   Up About a minute (healthy)   0.0.0.0:8080->8080/tcp   owncloud-docker-server_owncloud_1
14f073af9bff        webhippie/redis:latest     "/usr/bin/entrypoint…"   About a minute ago   Up About a minute (healthy)   6379/tcp                 owncloud-docker-server_redis_1
7b4fd28ee372        webhippie/mariadb:latest   "/usr/bin/entrypoint…"   About a minute ago   Up About a minute (healthy)   3306/tcp                 owncloud-docker-server_db_1
```

if all is working, you should be able to access the owncloud web site address now at the hostname or ip address of your docker server on port 80, for example :

```
http://192.168.83.110:8080
```

