---
id: z6v71dde35yunu57jz2ox9a
title: Errors
desc: 'Some common errors that I had that took me longer than 10 minutes to google or figure out'
updated: 1675780700679
created: 1675678907989
---

`docker.credentials.errors.InitializationError: docker-credential-desktop.exe not installed or not available in PATH` - After uninstalling docker desktop, solved by removing the config.json from `/.docker/config.json` which just contained [[Github issue | https://github.com/docker/for-win/issues/12355#issuecomment-642915975]]
```json
{
  "credsStore": "desktop.exe"
}
```
Which was running a check to find the desktop application.


```bash
ERROR: The Compose file './docker-compose.yml' is invalid because:
Unsupported config option for volumes: 'pgadmin_vol'
Unsupported config option for services: 'postgres_11_dev'
```
I added the version tag to the docker-compose.yml.

```bash
docker-compose up
ERROR: Version in "./docker-compose.yml" is unsupported. 
You might be seeing this error because you're using the wrong 
Compose file version. 
Either specify a supported version (e.g "2.2" or "3.3") and place your service definitions under the `services` key, 
or omit the `version` key and place your service definitions at the root of the file to use version 1.
```
Changed `version: '3.8'` to `version: '3.7'`

Also with using `FROM tomcat:latests` meant that when spooling it all up there was a `webapps` and `webapps.dist` directories where my WAR was stored in webapps and everything else was in `webapps.dist` this caused issues, resolved by dropping version down.

```
Unable to connect to server: connection to server at 'postgres_11_dev' (172.22.0.4), port 5432 failed: Connection refused
Is the server running on that host and accepting TCP/IP connections?
```
This issues seemed to be caused from 2 postgres instances running from the same port, resolved by forcing one to rebuild from scratch and a different port.

### Copy vs Add in Dockerfile