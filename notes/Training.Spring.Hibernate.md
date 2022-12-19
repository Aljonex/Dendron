---
id: 1hiprj6ql6x1q9hwj3g5aam
title: Hibernate
desc: ''
updated: 1671468651931
created: 1670861066022
---
Hibernate is an ORM (Object-Relational Mapping) tool, provides a framework to allow OOP to a relational database.
Effectively map Java objects to a database like SQL.

[[Community Docs | https://docs.jboss.org/hibernate/orm/3.5/reference/en/html/index.html]]

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

### The Session
[[ Session docs | https://docs.jboss.org/hibernate/orm/3.5/javadocs/org/hibernate/Session.html#update(java.lang.Object)]]

### Criteria API
Alternative way of manipulating objects and in turn the data available in *Row-Based Database Management Systems* (RDBMS), it lets you build a criteria query object where filtration rules and logical conditions can be applied.<br>
[[See some examples | https://www.tutorialspoint.com/hibernate/hibernate_criteria_queries.htm#:~:text=Hibernate%20provides%20alternate%20ways%20of,filtration%20rules%20and%20logical%20conditions.]]

```java
Criteria cr = session.createCriteria(Employee.class);

// To get records having salary more than 2000
cr.add(Restrictions.gt("salary", 2000));

// To get records having salary less than 2000
cr.add(Restrictions.lt("salary", 2000));

// To get records having fistName starting with zara
cr.add(Restrictions.like("firstName", "zara%"));

// Case sensitive form of the above restriction.
cr.add(Restrictions.ilike("firstName", "zara%"));

// To get records having salary in between 1000 and 2000
cr.add(Restrictions.between("salary", 1000, 2000));

// To check if the given property is null
cr.add(Restrictions.isNull("salary"));

// To check if the given property is not null
cr.add(Restrictions.isNotNull("salary"));

// To check if the given property is empty
cr.add(Restrictions.isEmpty("salary"));

// To check if the given property is not empty
cr.add(Restrictions.isNotEmpty("salary"));
```