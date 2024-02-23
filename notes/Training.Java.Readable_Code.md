---
id: rhs9wkyf3qm2e75crlnzgcw
title: Readable_Code
desc: ''
updated: 1708698846342
created: 1708620447706
---
First part of the Java pathway on Pluralsight

## Clean and Maintainable code
Generally if you can't look at code and understand what it's doing, there's a chance you simply lack the experience, but also good programmers will write code that is understandable at a high level due to good naming.

> "The best programs are written so computing machines can perform them quickly and so that human beings can understand them clearly" - Donald Knuth

Bad code means technical debt, increases time reading code, increases bugs, decreases job satisfaction.

> "*Always code as if the guy who ends up maintaining your code will be a violent psychopath who knows where you live*" <br>**THE MAN IN THE CAVE**

### Naming matters
- **Class** - noun (regardless of if concrete or abstract)
    - SRP, single responsibility principle - one reason to change
    - Avoid abstract suffixes like **manager** have a look at patterns i.e. *Builder, Singleton, Factory* 
        - Factory - builds objects
        - Builder - method chaining
- **Variable** - ideally one two words, specific, booleans have "is" as prefix, camelCase, or ALL_CAPS for constants
- **Methods** - if you have to look inside the method to know what it does, it can probably be named better
    - If method does more than name says, its a side effect (move it to another method)