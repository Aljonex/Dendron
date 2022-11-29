---
id: 2szyi8edbxk8l05x2kxhl5k
title: Modifiers
desc: ''
updated: 1669739164337
created: 1668420806765
---
[[Oracle docs | https://docs.oracle.com/javase/tutorial/java/javaOO/accesscontrol.html]]
## Keyword modifiers
The class modifier overwrites the inner modifiers.

### Public
- **Class**: All code can access the class, regardless of where it is.
- **Method**: All code can access the methods.
- **Attribute**: All code can access the variables.

### Final
- **Class**: Cannot be extended, i.e. the String class, if overriding this you would get unpredictable behaviour as it is used everywhere, therefore this is unacceptable.
- **Method**: Cannot be overriden, useful when the class doesn't need to be final but the method does.
- **Attribute**: Cannot be reassigned, are permanent. Useful for values that need to be consistent and immutable.

### Private
- **Class**: Can't have this but by default a java class is package private
- **Method**: Only code in the same class can access this method
- **Attribute**: Only code in the same class can access this variable, usually accessed via setters/getters

### Synchronized
Useful in multithreaded applications to stop race conditions. *Race conditions* are the condition of a program where its behaviour depends on relative timing or interleaving of multiple threads or processes.
*Thread-safe* is the term used to describe a program, code, data structure free of *race conditions* when accessed by multiple threads. 

A piece of logic marked with *synchronized* becomes a synchronized block, allowing only one thread to execute at a given time. The keyword works on **instance methods, static methods and code blocks**.
- **Class**:
- **Method**: Instance methods are synchronized over the instance of the owning class, so only one thread per instance of class can execute this method. If the method is static, **only one Class object exists per JVM per class** so only one thread can execute irrespective of instances.
- **Attribute**: (code blocks not variables) -> sometimes we only want to synchronize some instructions inside a method.

### Static
- **Class**: Can have nested static classes, static inner classes can only access static members of the outer class. Good as groups classes that'll only be used in one place (increases encapsulation). Increases readability by bringing code closer to only place its used, also more maintainable.
- **Method**: To perform operation that's not dependent upon instance creation. Share code across all instances. Good for manipulation of static variables or methods that don't depend on objects.
- **Attribute**: One instance is created and shared across all instances of the class, **stored in the heap memory**. Useful when value is independent of objects and meant to be shared. Reference these using class name not instance name.
```Java
public class Car {
    private String name;
    private String engine;
    
    public static int numberOfCars;
    
    public Car(String name, String engine) {
        this.name = name;
        this.engine = engine;
        numberOfCars++;
    }

    // getters and setters
}
```

### Protected
- **Class**: Can't have this but by default a java class is package private
- **Method**: Accessible in same package and subclasses.
- **Attribute**: Accessible in same package and subclasses.

### Final
If blank can only be initialised in the constructor.
Cannot change the value of it otherwise, it is constant and so the naming convention is `ALL_CAPS_WITH_UNDERSCORES`.

Final methods cannot be overridden.

Final classes cannot be extended.