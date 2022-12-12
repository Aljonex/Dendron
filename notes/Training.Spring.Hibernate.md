---
id: 1hiprj6ql6x1q9hwj3g5aam
title: Hibernate
desc: ''
updated: 1670861224968
created: 1670861066022
---


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