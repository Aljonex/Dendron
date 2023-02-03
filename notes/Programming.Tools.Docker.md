---
id: hpshftmtgv63cmozfu7fajq
title: Docker
desc: 'A notes page revolving around Docker as a tool and hopefully leading into the progression of information about Kubernetes, Helm, then Argo'
updated: 1675441793310
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

Docker allows developers to make use of *containers* to **Build, Share, & Run** applications where containers are:
> The packaging of software into standardised units for development, shipment, deployment. Making other devs lives easier as you are quickly able to deploy a application with default configuration already achieved removing time taken to set up and configure the same thing multiple times. Similar to saving a `venv` and using it to aid deployment of python apps you can package all code and dependencies up, containers *isolate* software from its environment and ensure that it works in a uniform manner despite differences for instance between development and staging.

It was started in 2013 after concept by Solomon Hykes in 2010, who now has moved onto the creation of *dagger io* which is a method of coding your CI/CD pipelines instead of using YAML. It is an engine that sits on a docker compatible runtime.

Form of virtualisation but unlike virtual machines, resources are **shared with the host directly** allowing lots of containers vs maybe a few VMs, this is because VMs have to quarantine off resources then **boot** an OS and communicate with host OS via translator called a **hypervisor**.

Begin with a DockerFile which instructs how the image will be built, from here you can add commands like `run`, then `docker build -t <nameOfDockerImage> ./fileLocation`, via `docker images` you can see all of your docker images. 
You can get docker images with `docker pull imageName` and then `docker build imageName`, if you want several containers to run and all be controlled by one thing this is done via `docker compose`.

`docker ps` shows running processes.

The problem solved by docker is the inability to run other applications written by devs due to versioning mismatches, it implements the concept of containerisation via `cgroups` and `namespaces` that isolate software from the infrastructure, OS and hardware of a given machine. 
It allows generalisation so applications can run natively on any machine and takes far fewer resources than virtual machines as works with the OS of the computer.

### Docker compose, Docker run, Kubernetes
`Docker run` is entirely command line based whereas docker-compose is a tool that reads configuration data from a YAML file. Docker run can only start one container at a time however `docker-compose` can configure and run multiple.

This leads us into Kubernetes, like `docker-compose` it is also a framework for container orchestration but the core difference is that *Kubernetes* runs containers across multiple computers (virtual or physical) whereas *Docker Compose* runs containers on a single host machine.


