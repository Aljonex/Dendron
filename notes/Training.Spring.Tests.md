---
id: 7kfpjaxw4fc3g4broh2e78k
title: Tests
desc: ''
updated: 1670948699895
created: 1670499446844
---
### Spring Test Annotations
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = { "classpath*:/spring/<file>.xml" })
```

### Testing with Spring
```Java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = { "classpath*:/spring/applicationContext.xml" })
public class HibernateIT {

    @Resource(name = "sessionFactory")
    private SessionFactory sessionFactory;

    @Rule
    public JUnitSoftAssertions softly = new JUnitSoftAssertions();

    private Session getSession() {
        return sessionFactory.getCurrentSession();
    }

    @Test
    @Transactional
    public void createVehicle() {
        Vehicle vehicle = new Vehicle("Hyundai", "i10", "Korea", "hyu-i10", 30);

        getSession().save(vehicle);
        
        softly.assertThat(getSession().contains(vehicle))
                .as("Checking the created vehicle is in the session")
                .isTrue();
    }

}
```
Now this is assuming that you have an `applicationContext` which inside is importing a hibernate config file (I named `hibernateContext`), which allows the use of the sessionFactory which was defined in the `hibernate` xml.
```XML
<!-- This enables the use of the @Transactional annotation, which will be relevant for reusable testing and in later tutorials -->
    <tx:annotation-driven transaction-manager="transactionManager" />

    <bean id = "transactionManager" class="org.springframework.orm.hibernate4.HibernateTransactionManager">
        <property name="sessionFactory" ref="sessionFactory"/>
    </bean>
```

### @Transactional
This is an annotation used that allows us to make use of a sessionFactory in the test, if you apply it to class level then all methods inside will be part of the same transaction. 
If you do it on each method then the methods themselves will be singular transactions, meaning anything else gets rolled back post-test.
Otherwise it'll rollback (at class level) at the end of all of the tests.
> [[Transactional and how it works | https://stackoverflow.com/questions/62157806/how-does-transactional-work-on-test-methods]]

### Spring Beans Scope
When creating bean definition you create a *recipe* for creating actual instances of the class defined by that bean definition. You can control the *scope* of the objects created from a particular bean definition.
 
 Scope | Definition
---------|-----------
singleton   | single instance of the bean, all requests will return same object (cached), modifying the object is reflected in all refs to that bean (this is default) (use singleton annotation and add scope parameter in xml)
prototype   | returns different instance each time its requested

The other 4 are only available in web-aware application context
- request      
- session      
- application  
- websocket    

### Profiles
Core feature of the framework allowing bean mapping to different profiles i.e. *dev, test, prod* you set this with `@Profile('<profile>')` annotation and can use either the name or the not operator to define when the component is used.

### init() method
Sometimes when spring beans are created devs are required to execute initialisation operations and cleanup operations before destroying the bean.
`init()` method is utilised to declare a custom method for the bean that is the initialiser and is the `init-method=""` spring <bean> tag.


### Session class
The below methods are used to store data in the database and make transient objects persistent in Hibernate.
- `save()` : generates new identifier and INSERT record into database, returns [[Serializable | http://javarevisited.blogspot.sg/2011/04/top-10-java-serialization-interview.html]] identifier
- `saveOrUpdate()` : INSERT or UPDATE based on the existence of a record
- `persist()` : return type is `void`, doesn't guarantee that identifier will be assigned immediately (may happen at flush time), works better at long-running conversation with extended Session context