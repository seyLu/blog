---
title: Docker Cheatsheet
description: "Docker stuff I learned from Nana Janashia"
date: "2024-02-02"
categories: ["Notes"]
tags: ["docker", "infra", "devops"]
image: Mitsuha_Miyamizu_holding_Modern_Operating_Systems.png
---

## Image vs Container

Image --> blueprint, not running

Container --> the one that you run

## Pull image from docker hub

`docker pull <image>`

## List running containers

`docker ps`

## List all containers

`docker ps -a`

## Start container

`docker start <container-hash>`

## Pull image + start container

`docker run <image>`

## Run container in detached mode with custom name

`docker run -d --name <custom-image-name> <image>`

## Run container in detached mode, with port mapping and custom name

`docker run -d -p <host-port>:<container-port> --name <image>`

## List all available networks

`docker network ls`

## Create a new docker network

`docker network create <network-name>`

## Connecting mongo and mongo-express on same network

### Via Docker Run Command

see [mongo docker docs](https://hub.docker.com/_/mongo):

```bash
docker run -d \
-p 27017:27017 \
-e MONGO_INITDB_ROOT_USERNAME=admin \
-e MONGO_INITDB_ROOT_PASSWORD=password \
--net mongo-network \
--name mongodb \
mongo
```

see [mongo-express docker docs](https://hub.docker.com/_/mongo-express):

```bash
docker run -d \
-p 8081:8081 \
-e ME_CONFIG_MONGODB_ADMINUSERNAME=admin \
-e ME_CONFIG_MONGODB_ADMINPASSWORD=password \
--net mongo-network \                  # same network as mongo
--name mongo-express \
-e ME_CONFIG_MONGODB_SERVER=mongodb \  # point to mongo container name
mongo-express
```

### Via Docker Compose

(Will take care of Setting Up Same Network Configuration)

<!-- prettier-ignore-start -->
```docker-compose.yaml
version: '3'
services:
  mongodb:
    image: mongo
    ports:
      - 27017:27017
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=password
  mongo-express:
    image: mongo-express
    ports:
      - 8081:8081
    environment:
      - ME_CONFIG_MONGODB_ADMINUSERNAME=admin
      - ME_CONFIG_MONGODB_ADMINPASSWORD=password
      - ME_CONFIG_MONGODB_SERVER=mongodb
```
<!-- prettier-ignore-end -->

`docker compose -f <path/to/yaml/file> up`
`docker compose -f <path/to/yaml/file> down`

## After pushing to remote via git, CI will build image and package it, and uses Dockerfile as blueprint

```Dockerfile
# FROM <image>
FROM node

# (alternative) better to set env in docker compose
ENV MONGO_DB_USERNAME=admin \
    MONGO_DB_PWD=password

# RUN <any_linux_command>
# you can have multiple RUN commands, but only one entry point
RUN mkdir -p /home/app

# COPY executes on host :: COPY <source_in_host> <dest_in_container>
COPY ./app /home/app

# ~cd
WORKDIR /home/app

# entry point command (run inside container)
CMD ["node", "server.js"]
```

### building a new docker image

`docker build -t <tag-name>:<tag-version> </path/to/Dockerfile>`

### list all available images

`docker images`

NOTE: When you adjust `Dockerfile`, you have to rebuild the image

## Deleting a Docker Image

### get container id

`docker ps -a | grep <image>`

### delete docker container

`docker rm <container-id>`

### get image id

`docker images | grep <image>`

### delete docker image

`docker rmi <image-id>`

## EXEC on docker container

```bash
# /bin/bash
# /bin/sh for containers with no bash (eg. base image is node)
docker exec -it <container-id> <path/to/shell>

# and to show env variables set in container
env
```

## Image naming in Docker registries

`registry_domain/image_name:tag`

```bash
# eg:
docker pull mongo:4.2
# is shorthand for
docker pull docker.io/library/mongo:4.2

# in AWS ECR it would look something like
docker pull 698767896.dkr.ecr.eu-central-1.amazonaws.com/my-app:1.0
```

## Pushing image to a different repo aside from Docker Hub

```bash
# tell docker which repo to push image
docker tag <image>:<tag> <registry_domain>/<image>:<tag>

docker push <registry_domain>/<image>:<tag>
```

## Volumes

`docker run -v <host_path>:<container_path>`

## Volume types

```bash
# host volume
docker run -v /home/mount/data:/var/lib/mysql/data

# anonymous volume
docker run -v /var/lib/mysql/data
# which docker will automatically create
# /var/lib/docker/volumes/<random_hash>/_data
# which then gets mounted to container

# named volume
docker run -v <name>:/var/lib/mysql/data
# improvement of anonymous volume
# reference volume by name
# should be used in production
```

```bash
# to find the path where db stores data, just search internet
# each db has different path, and common ones include:

# mongodb
/data/db

# mysql
/var/lib/mysql

# postgres
/var/lib/postgresql/data
```

<br>

<!-- prettier-ignore-start -->
```docker-compose.yaml
version: '3'
services:
  my-app:
    image: seylu/private:nana-docker_my-app-1.0
    ports:
      - 3000:3000

  mongodb:
    image: mongo
    ports:
      - 27017:27017
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=password
    volumes:
      - mongo-data:/data/db

  mongo-express:
    image: mongo-express
    ports:
      - 8081:8081
    environment:
      - ME_CONFIG_MONGODB_ADMINUSERNAME=admin
      - ME_CONFIG_MONGODB_ADMINPASSWORD=password
      - ME_CONFIG_MONGODB_SERVER=mongodb

volumes:
  mongo-data:
	driver: local
```
<!-- prettier-ignore-end -->

## Docker best practices

### 1. Use official Docker images as Base Image

```Dockerfile
# instead of installing base operating system image
# and installing packages by yourself
FROM ubuntu

RUN apt-get update && apt-get install -y node \
    && rm -rf /var/lib/apt/lists/*
```

```Dockerfile
# it is better to use the official 'node' image as base image
# which is already built with the best practices
FROM node
```

### 2. Fixate the version

```Dockerfile
# instead of using the latest tag (unpredictable)
# which might get a different image version and break stuff
FROM node
```

```Dockerfile
# fixate the version and use
# the more specific, the better
FROM node:17.0.1
```

### 3. Use Small-sized official images

```Dockerfile
# don't use full blown os images
# which contains a lot of features you will not use
# and larger surface area for attackers and known vulnerabilities
# and also, instead of
FROM node:17.0.1
```

```Dockerfile
# use image based on a leaner and smaller os distro
# alpine is a lightweight linux distro, and security oriented
# if app doesn't require any specific utils, choose leaner and smaller image
FROM node:17.0.1-alpine
```

### 4. Optimize Caching Image layers

```Dockerfile
# what are image layers?
# docker images are built based on a Dockerfile
# in Dockerfile, each command creates an image layer
# you can view image layers via
docker history <image>:<tag>

# docker caches each layer, saved on local filesystem

# if nothing has changed in a layer (or any layers preceding it),
# it will be reused from cache

# once a layer has changed, all following layers are re-created as well

# when app changes:
FROM node:17.0.1-alpine                       [cached]

WORKDIR /app                                  [cached]

COPY myapp /app                               [not cached]

RUN npm run install --production              [not cached]

CMD ["node", "src/index.js"]                  [not cached]
```

```Dockerfile
# optimize caching for 'npm install' layer
# does not re-run when project file changes

# when app changes
FROM node:17.0.1-alpine                       [cached]

WORKDIR /app                                  [cached]

COPY package.json package-lock.json .         [cached]

RUN npm run install --production              [cached]

COPY myapp /app                               [not cached]

CMD ["node", "src/index.js"]                  [not cached]

# will re-run only when package.json changes

# !!! Order Dockerfile commands from least to most frequently changing
```

### 5. Use .dockerignore file

```bash
# we don't need everything inside the docker image
#  auto-generated folders (eg. target, build)
#  README files
#  etc.

# use .dockerignore to
#  reduce image size
#  prevent unintended secrets exposurestart

# sample .dockerignore:
# ------------------- #

# ignore .git and .cache folders
.git/
.cache/

# ignore all markdown files
*.md

# ignore sensitive files
private.key
settings.json
```

### 6. Make use of Multi-stage Builds

```Dockerfile
# contents that you NEED for Building The Image
# but DONT NEED in the Final Image to Run The App
# includes:
#  arificats needed to compile app (eg. Java build tools)
#  dev tools/build tools
#  test dependencies
#  temporary files
# which causes increased image size and attack surface
# simple example of this is package.json or pom.xm
# which specifies project dependencies and are needed to install dep
# however, once image is installed, we don't need these files to run app


# using muli-stage build
# -------------------- #

# build stage
FROM maven AS build

WORKDIR /app

COPY myapp /app

RUN mvn package

# run stage
FROM tomcat
COPY --from=build /app/target/file.war /usr/local/tomcat/...

# -------------------- #
# each FROM instruction starts a new build stage
# you can selectively copy artifcats from one stage to another

# only the last Dockerfile commands are the image layers
```

### 7. Use the Least Privileged User

```Dockerfile
# default is root which is a security bad practice
# container could potentially have root access on docker host
# easier privilege escalation for an attacker

# create a dedicated user and group
# set required permissions
# change to non-root user with USER directive

# create group and user
RUN groupadd -r tom && useradd -g tom tom

# set ownership and permissions
RUN chown -R tom:tom /app

# switch to user
USER tom

CMD ["node", "index.js"]
```

```Dockerfile
# some Base Images have a generic user bundled in
# eg. node

FROM node:17.0.1-alpine

...

# set ownership and permissions
RUN chown -R node:node /app

# switch to ndoe user
USER node

CMD ["node", "index.js"]
```

### 8. Scan for Security Vulnerabilities

```bash
# deprecated
docker scan <image>:<version>

# use
docker scout quickview <image>:<version>
```
