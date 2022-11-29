---
id: 2ug7vsedxvwmtr1gruh8whn
title: Mockito
desc: ''
updated: 1669714473006
created: 1669134192062
---
### Learning objectives
- What is mocking? Why is it useful?
- How to mock a Java object?
- How to use mocks in [[JUnit | Training.JUnit]] tests?
- What is a spy? Why is it useful?
- [[Training.Mockito.BestPractice]]

**Mocking** = simulating external dependencies of code being tested, with either fake data or fake interactions.

When to mock?<br>
Pros:
- Less setup (doesn't come with dependencies)
- No cleanup - not saving anything
- Faster
- Isolated - tests pass without need for any dependencies to work

Cons:
- Less confidence - not using real interactions

### Spy vs Mock
When *Mockito* creates a mock - it does from the Class of a Type, not from the actual instance.
The Mock creates a bare-bones shell instance of the Class, instrumented track interactions with it.
A *Spy* however wraps an existing instance, it'll behave like a normal instance but it will also be instrumented to track all interactions with it.
```Java
@Test
public void whenCreateMock_thenCreated() {
    List mockedList = Mockito.mock(ArrayList.class);

    mockedList.add("one");
    Mockito.verify(mockedList).add("one");

    assertEquals(0, mockedList.size());
}

@Test
public void whenCreateSpy_thenCreate() {
    List spyList = Mockito.spy(new ArrayList());
    spyList.add("one");
    Mockito.verify(spyList).add("one");

    assertEquals(1, spyList.size());
}
```

### Useful Links
- [[Mockito Best Practices | https://mestachs.wordpress.com/2012/07/09/mockito-best-practices/]] 
- [[ Purpose of mock objects | https://stackoverflow.com/questions/3622455/what-is-the-purpose-of-mock-objects/3623574#3623574]]
- [[Hamcrest Matchers | http://hamcrest.org/JavaHamcrest/tutorial]]
- [[Spy vs Mock | https://stackoverflow.com/questions/15052984/what-is-the-difference-between-mocking-and-spying-when-using-mockito]]