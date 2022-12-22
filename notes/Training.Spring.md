---
id: v0ohtlg0wqyq2b74eulyw5z
title: Spring
desc: ''
updated: 1671552234851
created: 1670413716922
---
> Learning objectives
- What is dependency injection?
- Why do we like it?
- How does Spring do it?
- Why does this make design and testing easier?

Spring has a core feature of their framework, an *Inversion of Control* container (IoC), that can construct object of selected class and inject dependency objects via constructor, property, or function at execution time and dispose of them at a suitable time. 
This helps with creation and management of objects manually.

In a Spring application, an *application context* loads bean definitions and wires them together.
The application context is responsible for creation of and wiring of objects that make the application.

An example might be `ClassPathXmlApplicationContext` which as a context loads Spring context from one of more XML files located in application's classpath.

### Some application contexts
- `AnnotationConfigApplicationContext` - Loads spring app context from one or more java-based config classes
- `AnnotationConfigWebApplicationContext` - Loads spring web app context from one or more Java-based config classes
- `ClassPathXmlApplicationContext` - Loads context definition from one or more XML files located in classpath, treating context-definition files as class-path resources
- `FileSystemXmlApplicationContext` - Loads context definition from one or more XML files in the filesystem

i.e. `ApplicationContext context = newFileSystemXmlApplicationContext("c:/knight.xml");`
- `XmlWebApplicationContext` - Loads context definitions from one or more XML files contained in web application


### ApplicationContext.xml
The <property\> element does for property setter methods what the <constructor-arg\> element does for constructors.
In `constructor-arg` you use the `ref=""` attribute. 
However, in property you would use `name="" ref=""` where *name* is the name of the setter method, if there is only one you can add `@Autowired` above the relevant method if you wish.

See Chapter **2.4** of *Spring In Action Fourth Edition*