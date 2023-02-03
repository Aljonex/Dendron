---
id: pwpfcycntxea7oa7imuisej
title: Getting-Started-Tut
desc: ''
updated: 1675356985968
created: 1675350369987
---
### Getting started app
`docker build -t getting-started` gets errors when running on corporate proxies ([[as of Dec 2 2022 | https://github.com/docker/getting-started/issues/56]]). Switching to *SA Personal* from *SGS Staff* fixed this.


When updating a docker image, when changes have been made as required you should again run `docker build -t <name-of-image>`
But if you run the `docker run -dp <portnum>:<portnum> <image-name>` it'll fail due to having 2 different ID's for same image name.
Stop a container from CLI.
```bash
docker ps ###shows docker processes

docker stop <container-id> ###stops running container

docker rm <container-id> ###removes it from docker
```

You can execute commands on a running instance of docker to see the files it has 
```bash
 docker ps
CONTAINER ID   IMAGE             COMMAND                  CREATED         STATUS         PORTS                    NAMES
e934c32d1823   getting-started   "docker-entrypoint.sÔÇª"   4 minutes ago   Up 4 minutes   0.0.0.0:3000->3000/tcp   zealous_lichterman
d
alexjones3@ITEM-S134843 MINGW64 ~/Training/docker-test-app/getting-started/app (master)
$ docker exec e934c32d1823d ls
Dockerfile
node_modules
package.json
spec
src
yarn.lock
```

Now we want to create a persistent database, so we first create a volume `docker volume create todo-db` but as the current container doesn't have a persistent volume we must `docker rm -f <container-id>`.
Then the tutorial asks to make use of
`docker run -dp 3000:3000 --mount type=volume,src=todo-db,target=/etc/todos getting-started` but this doesn't work and instead *getting-started* must go before mount otherwise it'll attempt to mount to the local file directory.
This cannot be run from gitbash, instead use the terminal running wsl or you're a cretin.

Data saved
`\\wsl.localhost\docker-desktop-data\data\docker\volumes`
aka
```json
[
    {
        "CreatedAt": "2023-02-01T16:53:43Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/todo-db/_data",
        "Name": "todo-db",
        "Options": {},
        "Scope": "local"
    }
]
```


### Bind mounts vs Volume mounts
You can make a volume and mount it to persist data in a database across containers. But you can also make use of something called a *bind mount* which allows a directory to be shared from the host's filesystem into the container, then when working on an application you can mount source code onto a container which allows the container to see changes you make to the code immediately (as soon as a file is saved).

```bash
alexajones2@ITEM-S134843:/mnt/c/users/alexajones2/training/docker-test-app/getting-started/app (master) $ docker run -it --mount type=bind,src="$(pwd)",target=/src ubuntu bash
root@b91e5e33cb7f:/# pwd
/
root@b91e5e33cb7f:/# ls
bin  boot  dev  etc  home  lib  lib32  lib64  libx32  media  mnt  opt  proc  root  run  sbin  src  srv  sys  tmp  usr  var
root@b91e5e33cb7f:/#
```

The `--mount` option tells Docker to create a mount (type is bind) and *src* is the current working directory on the host machine, *target* is where the directory should appear in the container (`/src`)

Then running an app in dev container from WSL in terminal
```bash
alexajones2@ITEM-S134843:/mnt/c/users/alexajones2/training/docker-test-app/getting-started/app (master) $ docker run -dp 3000:3000 \
> -w /app --mount type=bind,src="$(pwd)",target=/app \
> node:18-alpine \
> sh -c "yarn install && yarn run dev"
```
- `-dp 3000:3000` is running detached (background) mode and creating a port mapping
- `-w /app` the working directory (directory command will run from)
- `--mount type=bind,src="$(pwd)",target=/app` bind mounts the current directory from host into /app directory in container
- `node:18-alpine` image to use
- `sh -c "yarn install && yarn run dev"`, the command, starting a shell and running `yarn install` to install packages and then `yarn run dev` to start development server, in `package.json` you can see that the `dev` script starts `nodemon`

`docker logs -f <container-id>` shows logs of the container, mine failed because I was on a corporate proxy.
If its working should see
```bash
docker logs -f 728e043c2f95
yarn install v1.22.19
[1/4] Resolving packages...
[2/4] Fetching packages...
[3/4] Linking dependencies...
[4/4] Building fresh packages...
Done in 50.75s.
yarn run v1.22.19
$ nodemon src/index.js
[nodemon] 2.0.20
[nodemon] to restart at any time, enter `rs`
[nodemon] watching path(s): *.*
[nodemon] watching extensions: js,mjs,json
[nodemon] starting `node src/index.js`
Using sqlite database at /etc/todos/todo.db
Listening on port 3000
```
Now for example changing line 109 of the app.js file from *Add Item* to *Add* and then refreshing localhost shows changes reflected. Once happy with changes made stop the container and rebuild it with `docker build -t getting-started`.

### Multi Container Apps
In general each container should **do one thing and do it well**, a couple of reasons:
- there's a high chance you'll have to scale APIs and front-ends differently than databases
- separate containers let you version and update versions in isolation
- you may use a container for the database locally, but may want to use a managed service for db in production
- multiple processes require a process manager (container only starts one process), this adds complexity to container startup/shutdown

**Containers** (by default) run in isolation and don't know about other processes or containers on same machine, how can we get them to talk to each other??
> ***NETWORKING*** - if 2 containers are on the same network, they can talk. Otherwise, they can't

To create a network:
`docker network create <network-name>`
```bash
alexajones2@ITEM-S134843:/mnt/c/users/alexajones2/training/docker-test-app/getting-started/app (master) $ docker run -d \
> --network todo-app --network-alias mysql \
> -v todo-mysql-data:/var/lib/mysql \
> -e MYSQL_ROOT_PASSWORD=secret \
> -e MYSQL_DATABASE=todos \
> mysql:8.0
```

To verify the sql database is up and running:
`sudo docker exec -it 9b6884331e64
 mysql -u root -p`
 Then enter the password as set before. (Executes an interactive TTY (unix terminal)) on the container and calls mysql and offers password entry.

 How do we know if another container is running as intended, how do we find it? Make uses of `nicolaka/netshoot` container, which has tools for troubleshooting or debugging networking issues.
 Start new container and connect it to same network
 `docker run -it --network todo-app nicolaka/netshoot`

 Then inside the container we can use the `dig mysql` command to lookup the IP of mysql. Had to spool up the mysql container before the node container using:
 ```bash
 docker run -dp 3000:3000 \
   -w /app -v "$(pwd):/app" \
   --network todo-app \
   -e MYSQL_HOST=mysql \
   -e MYSQL_USER=root \
   -e MYSQL_PASSWORD=secret \
   -e MYSQL_DB=todos \
   node:18-alpine \
   sh -c "yarn install && yarn run dev"
```
Or it errored out.
Then go to localhost:3000 and add some stuff to the todo list and to see it in the database you:
```bash
docker exec -it 9b6884331e64 mysql -p todos
Enter password:
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 10
Server version: 8.0.32 MySQL Community Server - GPL

Copyright (c) 2000, 2023, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> select * from todo_items
    -> ;
Empty set (0.00 sec)

mysql> select * from todo_items;
+--------------------------------------+----------------------+-----------+
| id                                   | name                 | completed |
+--------------------------------------+----------------------+-----------+
| 21b9bbce-aea7-4dfa-bc9f-e538c1601788 | arron                |         0 |
| 314997b4-b050-424d-9b51-af4adf653847 | very cool  very nice |         0 |
+--------------------------------------+----------------------+-----------+
2 rows in set (0.00 sec)

mysql>
```

### Docker Compose
Tool to help define and share multi-container applications, through the creation of a YAML file we can define the services with a single command, spin it all up or tear it all down. 
Keep the YAML at the root of project repo, to version control it, and easily enable others to contribute. Then just need to clone repo and compose the app.

Conversion of this command:
```bash
docker run -dp 3000:3000 \
  -w /app -v "$(pwd):/app" \
  --network todo-app \
  -e MYSQL_HOST=mysql \
  -e MYSQL_USER=root \
  -e MYSQL_PASSWORD=secret \
  -e MYSQL_DB=todos \
  node:18-alpine \
  sh -c "yarn install && yarn run dev"
```
To `docker-compose.yml`:
```yaml
docker-compose.yml
version: "3.7"

services:
  app: 
    image: node:18-alpine
    command: sh -c "yarn install && yarn run dev"
    ports:
      - 3000:3000
    working_dir: /app
    volumes: ./:/app
    environment:
      MYSQL_HOST: mysql
      MYSQL_USER: root
      MYSQL_PASSWORD: secret
      MYSQL_DB: todos
```

Then you must convert:
```bash
docker run -d \
  --network todo-app --network-alias mysql \
  -v todo-mysql-data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=secret \
  -e MYSQL_DATABASE=todos \
  mysql:8.0
```
And the final yaml will look like:
```yaml
version: "3.7"

services:
  app:
    image: node:18-alpine
    command: sh -c "yarn install && yarn run dev"
    ports:
      - 3000:3000
    working_dir: /app
    volumes: ./:/app
    environment:
      MYSQL_HOST: mysql
      MYSQL_USER: root
      MYSQL_PASSWORD: secret
      MYSQL_DB: todos
  mysql:
    image: mysql:8.0
    volumes:
      - todo-mysql-data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: todos

volumes:
  todo-mysql-data:
```
Make sure no other instances are running:
```bash
docker ps
CONTAINER ID   IMAGE            COMMAND                  CREATED             STATUS          PORTS                    NAMES
f26dc446fa7e   node:18-alpine   "docker-entrypoint.s…"   40 minutes ago      Up 40 minutes   0.0.0.0:3000->3000/tcp   quirky_curie
9b6884331e64   mysql:8.0        "docker-entrypoint.s…"   About an hour ago   Up 44 minutes   3306/tcp, 33060/tcp      eager_nightingale
alexajones2@ITEM-S134843:/mnt/c/Users/alexajones2/Training/docker-test-app/getting-started (master) $ docker rm -f f26dc446fa7e 9b6884331e6
4
f26dc446fa7e
9b6884331e64
```
Then start the stack with `docker compose up` (and -d flag to keep it in the background)

### Image building Best Practices
If logged into Docker Hub you can use `docker scan --login` then scan with `docker scan <image-name>`, you'll then get listed types of vulnerabilites and which version of relevant library fixes it.

` docker image history getting-started` shows the layers in the image created


In the app folder update Dockerfile:
```docker
# syntax=docker/dockerfile:1
 FROM node:18-alpine
 WORKDIR /app
 COPY package.json yarn.lock ./
 RUN yarn install --production
 COPY . .
 CMD ["node", "src/index.js"]
```
To copy package.json and yarn.lock into root, then after adding *node_modules* into `.dockerignore` you should be running the 2nd copy step so that it won't overwrite files created by RUN command.


Was getting error when building:
```bash
alexajones2@ITEM-S134843:/mnt/c/Users/alexajones2/Training/docker-test-app/getting-started/app$ docker build -t getting-started .
[+] Building 0.5s (2/3)
[+] Building 0.5s (3/3) FINISHED
 => [internal] load build definition from Dockerfile                                                                                  0.0s
 => => transferring dockerfile: 38B                                                                                                   0.0s
 => [internal] load .dockerignore                                                                                                     0.0s
 => => transferring context: 34B                                                                                                      0.0s
 => ERROR resolve image config for docker.io/docker/dockerfile:1                                                                      0.3s
------
 > resolve image config for docker.io/docker/dockerfile:1:
------
failed to solve with frontend dockerfile.v0: failed to solve with frontend gateway.v0: rpc error: code = Unknown desc = error getting credentials - err: exit status 1, out: ``
alexajones2@ITEM-S134843:/mnt/c/Users/alexajones2/Training/docker-test-app/getting-started/app$ docker pull docker/dockerfile:1
/usr/bin/docker-credential-desktop.exe: Invalid argument
1: Pulling from docker/dockerfile
dd092abd7f36: Already exists
Digest: sha256:d2d74ff22a0e47b21f4bbde337e2ac4cd0a02a2226ef79264878db3dc7e87df8
Status: Downloaded newer image for docker/dockerfile:1
docker.io/docker/dockerfile:1
alexajones2@ITEM-S134843:/mnt/c/Users/alexajones2/Training/docker-test-app/getting-started/app$ docker build -t getting-started .
[+] Building 11.5s (11/13)
```
Solved through forced pull of the docker/dockerfile:1 image.