---
id: 7kfpjaxw4fc3g4broh2e78k
title: Tests
desc: ''
updated: 1670500831905
created: 1670499446844
---
### Spring Test Annotations
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = { "classpath*:/spring/<file>.xml" })
```

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