---
id: 1iw2ure3mh0sbzfivh9znc2
title: Mocking
desc: ''
updated: 1669205655090
created: 1669135663121
---
A *mocked* object is just a fake object pretending to be real.
No beaviour associated by default, it doesn't do anything and will respond passively with no side effects to any method called on it. 
This is useful as tests can **ignore implementation details that don't concern what is being tested**

Imagine a hugely complex class, amd we have a new class that references the larger class when its in the system, we want to test they interact correctly. There are a couple of ways:
1. Assemble everything in the test <br>
Can create a new object of the big class, all of the DAOs and services it needs, and connect them all.
This will most likely take hundreds or thousands of lines and is *error prone*.
Most likely way to check is go into the database and check the message back is as we expect.
If any requirements change it'll need updating, we also have to hope that no one puts `@Rollback(false)` on the test.

2. Pull it out of context <br>
Pull the new class out of the wiring to test, this is better than the previous but has many of the same issues.
If anything about context changes aswell then changes must be made or reverted, this will lead to a jumble of *try - catch - finally* blocks all over tests.
The common theme of breaking is it has nothing to do with the class we're actually testing.

3. Mock it <br>
We can mock the complex class and pass it straight into the new version of the class we're testing.
Internally the test doesn't care how the complex class behaves,  it doesn't care what happens in the database.
That is for integration tests, and tests around the complex class.
We *only want to know that our new class interacts in the way we expect*.
```Java
NewClass newClass = mock(NewClass.class);
ComplexClass cc = new ComplexClass();
cc.setNewClass(newClass);
 
cc.performSomeActionThatDoesntReturnAnything(asset);
 
verify(newClass).amend(eq(asset.getAgreementInvoice()), eq(asset), eq(128), eq(true));
```
We can't test on return type of method, so instead we test on what it does to other objects. 
Here, we're making sure the method "*amend*" is called with the given arguments.
In the example, we wrapped the arguments in "`eq`", meaning a given argument must equal it to count as calling that method.
We can specify many complex criteria to 'match' valid arguments against, **using Hamcrest Matcher**.

If you import Mockito classes and do them statically it saves you using *Mockito.mock()* and can instead just use *mock()*.