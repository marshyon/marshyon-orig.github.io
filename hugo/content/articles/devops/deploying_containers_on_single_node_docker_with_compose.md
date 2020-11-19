---
title: "Deploying containers on single node docker nodes with compose"
date: 2020-11-13T12:32:58+01:00
authors: ["Jon Brookes"]
draft: false
---

When deploying containers in Docker, possibly for later use in orchestration platforms such as Swarm, K8s, OpenShift it is common to want to test run a docker container firstly on a local service such as a laptop, workstation or stand alone server hosting docker containers. 

It may be desirable to stop at this point and not to go to full cluster load balanced and high availability as is afforded by the likes of K8s as a 'one off' or 'private' implementation of an application does not necessarily need this degree of automation.

`docker-compose` files serve as a bridge between the one-off and full blown HA implementations and can be used to define single or multiple docker based containers within a package of applications each configured to work as a whole and as a step beyond single `Dockerfiles`.

Without this, we could use single Dockerfiles for which 'docker run' commands are marshalled with command lines and scripts. However this can lack consistency and does not lend itself as well to sharing and re-use as can a compose file.

### Prerequisites

* A working Docker server 
* docker-compose

### Example use of compose

An example of using `docker-compse` is to be found in this [gogs-private-repo](https://github.com/marshyon/gogs-private-repo) application.

It uses a single `docker-compose.yaml` file ( see [here](https://github.com/marshyon/gogs-private-repo/blob/master/docker-compose.yml) ) to define a working application that consists of 2 Docker containers which are each persisted locally mounted volumes :

```
version: "3.8"
services:
  repo:
    build: ./repo
    ports:
      - "3333:3333"
    volumes:
    - ./repo/data:/app/gogs/data
    - ./repo/repositories:/app/gogs/repositories
    - ./repo/custom/conf:/app/gogs/custom/conf   
    restart: always 
  webhook:
    build: ./webhookapp
    ports:
      - "4000:4000"  
    volumes:
    - ./webhookapp/scripts:/app/scripts
    environment:
      - SYSTEM_COMMAND="/app/scripts/do_something.sh"
    restart: always 
```    

Whilst relatively short in length, the above compose file defines several key features that would each lead to separate command line arguments fed to multiple `docker run` commands.

`services` : defines 1 or more docker container services, something similar to 'services' as a concept in other orchestration platforms.

There are here 2 such services, each with 

* `build` directories - which is the relative path to where each `Dockerfile` is located
* `ports` list - of 1 or more port mappings, the same as `-p` in `docker run` commands
* `volumes` list - of 1 or more volumes here mapped to local file storage
* `restart` flag to tell docker to start failing containers

`environment` used in the 2nd container is a list of 1 or more environment variables that are passed to the starting container - the same as `-e` in `docker run` commands.

This `docker-compose-yaml` file is created on a docker server with a single command :

```
docker-compose up -d
```

the `-d` switch tells the compose binary to run the containers in `detached` mode and to give back the terminal to the user rather than 'blocking' and waiting for <cntrl-c> to break the session.

```
docker-compose stop
```

will, as it suggests, stop the running containers and

```
docker-compose down
```

will shut down and remove the running containers together with any internal volumes or networks associated with this compose file.

> in this example a nice thing to get for free is that a network is automatically created by docker-compose for the current docker-compose.yaml file 



