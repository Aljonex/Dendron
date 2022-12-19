---
id: 1hiprj6ql6x1q9hwj3g5aam
title: Hibernate
desc: ''
updated: 1671206533465
created: 1670861066022
---
Hibernate is an ORM (Object-Relational Mapping) tool, provides a framework to allow OOP to a relational database.
Effectively map Java objects to a database like SQL.

### Configuration
When being used with Spring 

### Test failure
```bash
Dec 16, 2022 4:00:47 PM org.hibernate.engine.jdbc.spi.SqlExceptionHelper logExceptions
WARN: SQL Error: 0, SQLState: null
Dec 16, 2022 4:00:47 PM org.hibernate.engine.jdbc.spi.SqlExceptionHelper logExceptions
ERROR: Cannot create PoolableConnectionFactory (Connection refused. Check that the hostname and port are correct and that the postmaster is accepting TCP/IP connections.)
```
This is due to the SQL session not being started, as I'm using a docker container with Postgres I must go into my WSL (set up on terminal) `cd local-dev-compose/Postgres11` and then `docker-compose up`.