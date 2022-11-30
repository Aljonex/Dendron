---
id: 2rgyftzf1epf699xi09wibl
title: Tut1-5
desc: ''
updated: 1669825657242
created: 1669205701466
---
## All learning objectives
### Git #1
- Basics of Version Control
- How to use Git Bash
- Simple git commands
- Git Branching structure

### Project Setup #2
- Maven Dependency Management + Project Building Basics
- Intellij for editing class in a project
- What is BigDecimal and why use [[Training.Intellij.BigDecimal]]
- [[Training.Intellij.Modifiers]] : **static** lives at the top of the file

### JUnit #3
- Different kinds of test: *Unit, Integration, End to End*
- Differences between automated and manual testing
- Testing best practices: *.as("desc")* should always be before the check
- JUnit Annotations and Assertions
- MoreUnit
- How to create a test class
- How to test client Configuration
- How to calculate Code Coverage and why it is useful

Generally setup, teardown, and test specific methods live at the top of the suite.


### Mocking and Mockito #4
- What is mocking? Why is it useful?
- How to mock a Java object?
- How to use mocks in [[JUnit | Training.JUnit]] tests?
- What is a spy? Why is it useful?
- Difference between test double objects

Don't use spies because they're fragile and cause side effects, unless you have to!

### Logging #5
- Differences between log4j and slf4j
- Basics of logging framework configuration
- What logging levels are

## Extras
- Composition: A means of avoiding inheritance, involves a constructor instantiating the object that would have been extended. Make sure to make use of polymorphism i.e. if object you wanted was implementing an interface, use that interface to future proof for extra extensions.