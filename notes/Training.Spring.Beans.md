---
id: y1orx0atlwrvu0xam9tg8ru
title: Beans
desc: ''
updated: 1671535954068
created: 1671534482862
---
### Basic
```XML
<?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
        xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
        
        <bean id="knight" class="com.springinaction.knights.BraveKnight">
            <constructor-arg ref="quest" />
        </bean>
        <bean id="quest" class="com.springinaction.knights.SlayDragonQuest">
            <constructor-arg value="#{T(System).out}" />
        </bean>
    </beans>
```
The classes are declared as **beans** in Spring and in the case of `BraveKnight` bean, its *constructed, passing a reference to SlayDragonQuest bean as a constructor argument*
However, the above is constructor injection, which is generally looked upon a bit poorly as if the constructor needs many arguments it will quickly become repetetive, recommended currently (in Sopra) is the use of property injection.
![[Training.Spring.Injection#property-injection]]

The `SlayDragonQuest` bean declaration uses the *Spring Expression language* to pass `System.out` (which is a PrintStream) to `SlayDragonQuest`'s constructor.

### Abstract
Parent abstract bean definition is used only to group common properties, so you avoid repetition in XML.
```xml
<bean id="<name>" abstract="true">
```
Above doesn't even have to be a class but could have constant properties shared between beans so then you would put `parent="<name>"` in the beans that share it.<br>
[[ Stackoverflow answer | https://stackoverflow.com/questions/9397532/what-is-meant-by-abstract-true-in-spring]]