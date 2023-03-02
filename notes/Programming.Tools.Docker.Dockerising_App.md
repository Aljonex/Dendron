---
id: 99cxam6h1p8kzmfci3ags54
title: Dockerising App
desc: ''
updated: 1677580046710
created: 1677579933237
---
### DockerFile
```
# syntax=docker/dockerfile:1
FROM maven AS build
WORKDIR /app
COPY . .
CMD ["mvn", "clean", "package", "-Dskiptests"]

FROM tomcat:9.0.20-jdk8
COPY --from=build /app/target/*.war /usr/local/tomcat/webapps/aj.war
EXPOSE 8081
CMD ["catalina.sh", "run"]
```

### Docker Compose
```yaml
version: '3.7'

services:
  postgres_11_dev:
    image: postgres:11.8
    container_name: postgres_11_dev
    environment:
      POSTGRES_PASSWORD: 'welcome'
    ports:
      - '5432:5432'
    volumes:
      - postgres_11_vol:/var/lib/postgresql/data
    privileged: true
  pgadmin:
    container_name: pgadmin
    image: dpage/pgadmin4:latest
    environment:
      PGADMIN_DEFAULT_EMAIL: 'postgres@soprabanking.com'
      PGADMIN_DEFAULT_PASSWORD: 'welcome'
    ports:
      - '9000:80'
    volumes:
      - pgadmin_vol:/var/lib/pgadmin
      - pgadmin_vol:/certs/server.cert
      - pgadmin_vol:/certs/server.key
      - pgadmin_vol:/pgadmin4/servers.json
    restart: unless-stopped
  tomcat:
    build:
      context: .
      dockerfile: Dockerfile
    image: jsf-app
    ports:
      - '8081:8080'
    environment:
      - DB_USER=postgres
      - DB_PASSWORD=welcome
      - DB_HOST=postgres_11_dev
    depends_on:
      - postgres_11_dev
      - pgadmin
    restart: unless-stopped

volumes:
  pgadmin_vol:
  postgres_11_vol:
```

### Changes to hibernate context to allow use of env variables
```xml
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
        <property name="url" value="jdbc:postgresql://${env.DB_HOST ?: 'localhost'}:5432/"/>
        <property name="driverClassName" value="org.postgresql.Driver"/>
        <property name="username" value="${env.DB_USER ?: 'postgres'}"/>
        <property name="password" value="${env.DB_PASSWORD ?: 'welcome'}"/>
</bean>
```

#### Accessing web page
`localhost:8081/aj`