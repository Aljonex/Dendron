---
id: 28is9yzjhiyxa7shoa8i9d0
title: Tut6-8
desc: ''
updated: 1673439186239
created: 1670413655191
---
## All learning objectives
### Spring #6
- *What is dependency injection?*<br>
Dependency injection is a way of passing needed objects or functions to an object.
- *Why do we like it?*<br>
It removes strong coupling of objects, also makes code easier to test and reuse and reduces complexity of construction of lots of objects when unneeded.
- *How does Spring do it?*<br>
Spring performs DI through Spring beans, which are representations or references to classes.
- *Why does this make design and testing easier?* <br>
Being able to inject whatever dependency you want at compile time means that you can write less tests and don't have to initialise as many dependencies, it allows easy mocking.

*Beans* can inherit from a parent definition, much like Java and the child definition can override values or add more as needed. You can use the `parent=<beanName>` attribute in the child bean.

*Bean scope* is a definition of the life cycle and visibility of the bean in the contexts we use it. There are 6 types but the main are `singleton` and `prototype`, singleton being a single instance of the bean, all changes will be reflected in all references. Prototype returns a different bean on each reference.

Also useful: spring parent beans, spring beans scope, spring init() (`init-method` tag), profiles

### Hibernate 1
- *What is Hibernate?*<br>
An Object Relational Mapping tool which provides a framework to allow Object Oriented Programming to map to relational databases.
- *Why do we use it?*<br>
Reduction of lines of code by using Hibernate annotations to map objects to databases, also allows easy querying and return of Objects from database. Don't have to manually handle persistent data so development time is reduced.
- *What is the difference between Hibernate and JPA?*<br>
Hibernate is an implementation of JPA guidelines and helps mapping Java types to SQL types, JPA is a specification of rules and guidelines to set interfaces for implementing object-relational mapping. Hibernate is an open-source framework that is an ORM tool to simplify Java application interaction with the database.
- *3 states of entities* <br>
    - **Transient** - When you create an object but do not associate it with Hibernate session, then it is in Transient state. Any modifications to such object using setter methods doesn't reflect the change in the database.
    - **Persistent** - Here the object is attached to the Hibernate session. So now the Hibernate session manages this object. Any changes made to this object gets reflected in the database. Because Hibernate designed it in such way that, if any modifications is made to a Persistent object, it automatically gets updated in the database, when the session is flushed. (This is Hibernate's capability).
    - **Detached** - This state is similar to Transient. The only difference is that an object in detached state was previously in the session(i.e. in persistent state ). But now this is out of the session, because of either closing of the session or calling the evict(Object) method of session.

### Hibernate 2
- *Understand how data is added and retrieved from databases*<br>
Using either the `sessionFactory.getCurrentSession().save(entity)` or the `persist(entity)` but the difference is mainly that `save()` returns a Serializable object and `persist()` returns void.
- *What Criteria is and how to use it*<br>
One of the ways Hibernate offers manipulation of objects into Row-based Database Management System tables, an API letting you build a criteria query object and you can apply filtration rules and logical conditions. *Session* interface offers `createCriteria(Clazz.class)` method which creates a criteria object that returns instances of the persistence object's class when application executes criteria query. A simple example is depicted below (returning every object of Employee class):
```Java
Criteria cr = session.createCriteria(Employee.class);
List results = cr.list();
```
And using `add()` method you can add restrictions to the query.
![[Training.Spring.Hibernate#criteria-api]]
- *What HQL is and how to use it*<br>
*HQL* is **Hibernate Query Language** (similar to SQL) but is fully object-oriented and understands things like *inheritance, polymorphism, assosciation*, other than Java classes and prop and has all of the SQL queries (from, joins, associations, aggregate functions like average and sum and max/min).
An example is:
```
Query query = session.createQuery("select max(salary) from Emp")
List<Integer> list = query.list();
System.out.println(list.get(0));
```
- *Type of joins*:
    - `(Inner) JOIN` - matching values in both tables
    - `Left (Outer) JOIN` - all from left and matching from right
    - `Right (Outer) JOIN` - all from right and matching from left
    - `Full (Outer) JOIN` - all records when there's a match in either left or right
- *What SQL binding is and how to use it in HQL*<br>
SQL binding is a way to dynamically inject a variable into an HQL query (it is referred to as a *bind variable*), the named placeholder is preceded by a colon and the actual value is substitued at runtime using the `setParameter()` method, without binding we have String concatenation like this (less readable):<br>
`String hql = "from Student student where student.studentName = '" + studentName+ "'";`<br>
You can have named parameter binding or positional parameter (ignore the second as is less common and less user friendly)
```Java
String hql = "from Student student where student.rollNumber= :rollNumber";
Query query = session.createQuery(hql);
query.setParameter("rollNumber", "3");
List result = query.list();

String hql = "from Student student where student.studentName= :studentName";
Query query = session.createQuery(hql);
query.setString("studentName", "Sweety Rajput");
List result = query.list();

Student student= new Student();
student.setCourse("MCA");
String hql = "from Student student where student.course= :course";
Query query = session.createQuery(hql);
query .setProperties(student);
List result = query.list();
```
As you can see above there are 3 ways and HQL is very smart! It can detect parameters, a string, or detect object properties and match it with the binding variable (colon parameter)!!
- Usage of Projections<br>
Generally used in the context of some *Criteria*, projections are used to query a subset of attributes of entity/entities, its useful for aggregations and fetching only certain columns (thereby improving performance of the query)
```Java
Criteria criteria = getSession().createCriteria(Vehicle.class);
        criteria.setProjection(Projections.distinct(
                Projections.projectionList()
                        .add(Projections.property("make"), "make")
                        .add(Projections.property("model"), "model")
                        .add(Projections.property("derivative"), "derivative")
        ));
```
Above is an example of projection where choosing all unique combinations of the vehicles from their make, model and derivative.
- *Criteria vs HQL*<br>
    - HQL can perform select and non-select, but is vulnerable to SQL injections as queries are fixed or paramaterized, it is generally faster than Criteria however
    - Criteria dynamically generates queries so is safe from SQL injection
- *Transient*
    - What does it mean to mark something as transient?<br>
    It means its not associated with the Hibernate session so when you alter it via setter methods, it doesn't change anything in the Hibernate entities
    - What are the use cases?<br>
    `static` and `final` fields, otherwise you can use `@Transient` annotation, fields that are only useful for the backend logic.
- *Difference between `get` and `load`*
    - Key differences<br>
    `get()` loads data as soon as its called, `load()` loads data when required (supports lazy loading), however `load()` throws an exception when data isn't found so only use when we know data exists
    - Use cases for each<br>
    `get` is useful for making sure data exists in the database, get is *eager* loading and load is *lazy*, use `load()` if you're sure the object exists


### Topics from previous quizzes
- Autowired vs Resource annotation
- BOM
- Property vs constructor injection
- Normalisation
- Lazy loading in transactions
- Table vs Record
- Foreign key
- Database transaction
- Optimistic locking
- Cascading
- Alias