---
id: p6sm20au85linanf6g6m9jm
title: Injection
desc: ''
updated: 1670855592600
created: 1670854946555
---
## Dependency Injection
In Spring there's 2 types of dependency injection:
- Setter DI: the simpler of the 2, the DI will be injected via setter/getter methods, the property to be set is under the `<property>` tag
- Constructor DI: via constructors and using the `<constructor-arg>` tag

Both in the configuration xml.


### Property Injection
In the config.xml (`applicationContext.xml`) you can commonly inject properties via setter methods to keep things loosely coupled.

The naming convention for the name of the property however must be **exactly** what the name of the setter/getter methods are minus the *set* or *get*.
You must also explicitly define a default constructor (empty).

### Constructor Injection
For constructor injection you use the tag `constructor-arg` and can include the index of the arg as an attribute or simply list them in order, from there you must just reference the pre-defined bean. Provided it is of the correct type.


[[Example | https://www.geeksforgeeks.org/spring-dependency-injection-by-setter-method/#:~:text=Setter%20injection%20is%20a%20dependency,dependency%20by%20the%20setter%20method.]]

[[A bit on standards | https://stackoverflow.com/questions/21613703/spring-bean-property-xxx-is-not-writable-or-has-an-invalid-setter-method]]