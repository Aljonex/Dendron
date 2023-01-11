---
id: 1hiprj6ql6x1q9hwj3g5aam
title: Hibernate
desc: ''
updated: 1672933645829
created: 1670861066022
---
Hibernate is an ORM (Object-Relational Mapping) tool, provides a framework to allow OOP to a relational database.
Effectively map Java objects to a database like SQL.

[[ Community Docs | https://docs.jboss.org/hibernate/orm/3.5/reference/en/html/index.html]]

### Configuration
When being used with Spring we use a hibernate config file, which I called hibernateContext.xml and a Spring applicationContext.xml that allows wiring of beans (Spring) and setup of session factory for database connection with hibernate.

### @Transactional
There are 2 imports for transactional you can use:
`javax.transaction.Transactional` - This annotation will only apply to spring beans

`org.springframework.transaction.annotation.Transactional` - This annotation will apply to all *Contexts and Dependency Injection* beans. This was introduced in Spring framework 4.0.

### Test failure
```bash
Dec 16, 2022 4:00:47 PM org.hibernate.engine.jdbc.spi.SqlExceptionHelper logExceptions
WARN: SQL Error: 0, SQLState: null
Dec 16, 2022 4:00:47 PM org.hibernate.engine.jdbc.spi.SqlExceptionHelper logExceptions
ERROR: Cannot create PoolableConnectionFactory (Connection refused. Check that the hostname and port are correct and that the postmaster is accepting TCP/IP connections.)
```
This is due to the SQL session not being started, as I'm using a docker container with Postgres I must go into my WSL (set up on terminal) `cd local-dev-compose/Postgres11` and then `docker-compose up`.

### Entity States
**Transient** - When you create an object but do not associate it with Hibernate session, then it is in Transient state. Any modifications to such object using setter methods doesn't reflect the change in the database.

**Persistent** - Here the object is attached to the Hibernate session. So now the Hibernate session manages this object. Any changes made to this object gets reflected in the database. Because Hibernate designed it in such way that, if any modifications is made to a Persistent object, it automatically gets updated in the database, when the session is flushed. (This is Hibernate's capability).

**Detached** - This state is similar to Transient. The only difference is that an object in detached state was previously in the session(i.e. in persistent state ). But now this is out of the session, because of either closing of the session or calling the evict(Object) method of session.

### The Session
[[ Session docs | https://docs.jboss.org/hibernate/orm/3.5/javadocs/org/hibernate/Session.html#update(java.lang.Object)]]

### Annotations
When writing to nullable fields in JPA (Hibernate in JBoss) it is best practice to use wrapper types of primitives i.e. *int -> Integer* as these CAN have null as a value, and won't throw a null exception if the value isn't in the database.
However, you can specify (non-nullable) and use a primitive if you're careful as they are more performant.
These annotations are put on a POJO class to define the mapping of objects to the defined table.

- `@Entity` - declares the POJO class as an entity to be mapped to a table
- `@Table` - optional if you wish to rename the table with `(name="")`, otherwise it will use the unqualified java class name as the table name (basically just the class name unchanged)
- `@Id` - This is necessary for all entity beans otherwise they cannot be in the table, as databases require some way of uniquely identifying entities. Should be accompanied with the `@GeneratedValue` annotation, which can take *strategy* or *generator* parameters but will default as an `AUTO` strategy meaning on any insert operation it will auto increment in the database.
- `@Column` - specify deetails of column to which field or property will be mapped, most common attributes are *name, length, nullable, unique*

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

### What is HQL
HQL is the hibernate query language, it is extremely powerful and similar in appearance to SQL. 
Unlike SQL, however, it is object-oriented and understands notions like *inheritance, polymorphism, and association*, other than Java class and property names, queries are case-insensitive, [[see more on JBoss docs | https://docs.jboss.org/hibernate/core/3.3/reference/en/html/queryhql.html]]

Although HQL can perform both select and non-select operations, it is useful to bear in mind that Criteria is safe from SQL injection whereas HQL is vulnerable to it as the queries are fixed or paramaterized.

### SQL binding and use in HQL
SQL Binding or SQL injection and parameter binding in hibernate is a means of dynamically injecting values to a query at runtime.

Example:
```java
String hql = "from Student as student where student.rollNumber= :rollNumber";
Query query = session.createQuery(hql);
query.setParameter("rollNumber", "3);
List.result = query.list();
```
You can use `setParameter`, `setString`, `setProperties` to discover the data type, tell the parameter that its a string, or pass an object in (Hibernate check's its properties to match with the colon parameter).

### Use of Projections
Projections describe which columns you select from database and which form Hibernate provides them to you. 
In a JPQLquery its everything between *SELECT* and *FROM* keywords.

JPA and Hibernate support 3 groups of projections:
1. Scalar values
2. Entities
3. DTOs

### Transience

### get() vs load()

### Personal errors
Had an error saying that my relation didn't have the property as I had specified.
When going into the pgAdmin on localhost:9000 and through server, right click+Register>Server

Then in General, I named it what I wished, in connection I uesd 'Host name' of `postgres_11_dev` and then saved it. From here I could view my schemas and tables and column names, despite the sessionFactory having the column name I wished, there was a disparity with postgres, my fix was completely removing the docker volume and recomposing it.
See ![[Programming.Tools.Docker]]

The naming and setup in pgAdmin was due to 
```yml
version: "3.4"

services:
  postgres_11_dev:
    image: postgres-11-dev:latest
    environment:
      POSTGRES_PASSWORD: 'welcome'
    build:
      context: ./
    ports:
      - 5432:5432
    volumes:
      - postgres_11_vol:/var/lib/postgresql/data
  pgadmin:
    image: dpage/pgadmin4:latest
    environment:
      PGADMIN_DEFAULT_EMAIL: 'postgres@soprabanking.com'
      PGADMIN_DEFAULT_PASSWORD: 'welcome'
    ports:
            - 9000:80
    volumes:
      - pgadmin_vol:/var/lib/pgadmin
      - pgadmin_vol:/certs/server.cert
      - pgadmin_vol:/certs/server.key
      - pgadmin_vol:/pgadmin4/servers.json
```

Useful resources:
- **Java Persistence with Hibernate** Second Edition by *Christian Bauer, Gavin King, Gary Gregory*
- [[Hibernate 4.3 User Manual | https://docs.jboss.org/hibernate/orm/4.3/manual/en-US/html/]]
- [[Hibernate 4.3 Javadoc | https://docs.jboss.org/hibernate/orm/4.3/javadocs/]]