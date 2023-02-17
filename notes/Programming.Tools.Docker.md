---
id: hpshftmtgv63cmozfu7fajq
title: Docker
desc: 'A notes page revolving around Docker as a tool and hopefully leading into the progression of information about Kubernetes, Helm, then Argo'
updated: 1676644175446
created: 1671621660756
---
### Docker
Docker is a container tool that is open platform for *developing, shipping, and running applications* in a way that you want. 
These are set up in containers which have a `docker-compose.yml` which defines the services that make it up.
The app environment is defined in a `Dockerfile` so it can be reproduced anywhere.

### Docker build
Builds or rebuilds services, can use `docker build` or `docker-compose build`

### Connecting Postgres with Spring
[[Setting up env variables and using | https://stackoverflow.com/questions/3965446/how-to-read-system-environment-variable-in-spring-applicationcontext]]
<br>Changed from this:
```xml
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
        <property name="url" value="jdbc:postgresql://localhost:5432/"/>
        <property name="driverClassName" value="org.postgresql.Driver"/>
        <property name="username" value="postgres"/>
        <property name="password" value="welcome"/>
    </bean>
```
<br>To this:
```xml
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
        <property name="url" value="jdbc:postgresql://#{systemEnvironment['DB_HOST'] ?: 'localhost'}:5432/"/>
        <property name="driverClassName" value="org.postgresql.Driver"/>
        <property name="username" value="#{systemEnvironment['DB_USER'] ?: 'postgres'}"/>
        <property name="password" value="#{systemEnvironment['DB_PASSWORD'] ?: 'welcome'}"/>
    </bean>
```
With these environment variables in `docker-compose.yml`:
```yml
version: '3.7'

services:
  tomcat:
    build:
      context: .
      dockerfile: Dockerfile
    image: jsf-app
    ports:
      - '8081:8081'
    environment:
      - DB_USER=postgres
      - DB_PASSWORD=welcome
      - DB_HOST=postgres_11_dev
    networks:
      - default

networks:
  default:
    external:
      name: postgres-11_default

```
Where hostname was gathered from inspecting the container for postgres and near the bottom showed this as the network alias:

```json
"Networks": {
                "postgres-11_default": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": [
                        "a9d16effd0d7",
                        "postgres_11_dev"
                    ],
```

### Docker compose
`docker-compose up` creates and starts containers

`docker-compose down` stops and removes containers, networks

`docker exec -it <container-id_or_name> bash` runs the container's shell

`sudo lsof -i :<port-num>` is list of open files and ports and specifying IPv[4|6] files with given port number

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

### Build issue
For some reason `META-INF` wasn't appearing in the container folders when using this Dockerfile setup:
```Dockerfile
FROM maven:3.6.3-jdk-8 as maven_builder
WORKDIR /app
ADD pom.xml .
RUN ["mvn", "clean", "package", "-Dskiptests"]

FROM tomcat:9.0.20-jdk8
RUN sed -i 's/port="8080"/port="8081"/' ${CATALINA_HOME}/conf/server.xml
COPY --from=maven_builder /app/target/tutorial-alex-jones-11-SNAPSHOT.war /usr/local/tomcat/webapps/aj.war
COPY ./app/src/main/webapp /usr/local/tomcat/webapps/aj/
#ADD target/tutorial-alex-jones-11-SNAPSHOT.war /usr/local/tomcat/webapps/aj.war
EXPOSE 8081
CMD ["catalina.sh", "run"]
```
Something to do with copying from the src into the folder in the container as src doesn't have META-INF.

### Local Registry
This needs to be done for when using kubernetes unless you have pushed your custom images to the DockerHub (also a viable option).

So:
1. Start local registry container:<br> `docker run -d -p 5000:5000 --restart=always --name registry registry:2`
2. Do `docker images` to find out repository and tag of local image, then create new tag for local image `docker tag <local-image-repository>:<local-image-tag> localhost:5000/<local-image-name>`<br>
If the TAG for local image is `<none>` then<br>
`docker tag <local-image-repository> localhost:5000/<local-image-name>`
3. Push to local registry<br>
`docker push localhost:5000/<local-image-name>`<br>
Which will automatically add latest tag to the `localhost:5000/<local-image-name>` when can be checked again in `docker images`
4. In yaml file for Kubernetes set `imagePullPolicy` to `IfNotPresent`

`curl localhost:5000/v2/_catalog` to see what repositories are in your registry.