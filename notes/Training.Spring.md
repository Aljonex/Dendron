---
id: v0ohtlg0wqyq2b74eulyw5z
title: Spring
desc: ''
updated: 1670425923479
created: 1670413716922
---
> Learning objectives
- What is dependency injection?
- Why do we like it?
- How does Spring do it?
- Why does this make design and testing easier?

In a Spring application, an *application context* loads bean definitions and wires them together.
The application context is responsible for creation of and wiring of objects that make the application.

An example might be `ClassPathXmlApplicationContext` which as a context loads Spring context from one of more XML files located in application's classpath.

#### Some application contexts
- `AnnotationConfigApplicationContext` - Loads spring app context from one or more java-based config classes
- `AnnotationConfigWebApplicationContext` - Loads spring web app context from one or more Java-based config classes
- `ClassPathXmlApplicationContext` - Loads context definition from one or more XML files located in classpath, treating context-definition files as class-path resources
- `FileSystemXmlApplicationContext` - Loads context definition from one or more XML files in the filesystem

i.e. `ApplicationContext context = newFileSystemXmlApplicationContext("c:/knight.xml");`
- `XmlWebApplicationContext` - Loads context definitions from one or more XML files contained in web application