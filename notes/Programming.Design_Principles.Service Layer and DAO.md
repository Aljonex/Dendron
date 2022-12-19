---
id: rqnajawid7rne0h66gnqxku
title: Service Layer and DAO
desc: ''
updated: 1671459549379
created: 1671459245665
---
The DAO or *Data Access Object* is often paired with a service object, the 2 combined allow operations and business logic to be performed on a database.

The DAO should be as light as possible and simply provide a connection to the database, it can also be abstracted so different database backends are used or have an overall generic DAO implementation that can then be extended by children to follow the DRY principle and keep CRUD operations in one place.

The Service Layer is where any extra business logic on the data needs to be performed, where the data is retrieved from the DAO, it also adds security as the service layer has no relation to the database then its harder to access the database without the service.