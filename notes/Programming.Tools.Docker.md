---
id: hpshftmtgv63cmozfu7fajq
title: Docker
desc: ''
updated: 1671621936965
created: 1671621660756
---
Docker is a container tool that is open platform for *developing, shipping, and running applications* in a way that you want. 
These are set up in containers which have a `docker-compose.yml` which defines the services that make it up.
The app environment is defined in a `Dockerfile` so it can be reproduced anywhere.

### Docker build
Builds or rebuilds services, can use `docker build` or `docker-compose build`

### Docker compose
`docker-compose up` creates and starts containers

`docker-compose down` stops and removes containers, networks

### Docker volume
`docker volume` command

Can add options like ` list`, to show all of the volumes.
`rm <volName>` to remove the volume.