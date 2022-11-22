---
id: 2xtszsykwje2oqzlorzesnd
title: JUnitBasics
desc: ''
updated: 1668443612052
created: 1668440090975
---
This page will cover the first 3 learning objectives of the above page.
> Learning Objectives
- Testing best practices
- JUnit Annotations and Assertions
- MoreUnit

## Best Practices
1. Always test core methods
    - Almost impossible for 100% code coverage, write unit tests for methods likely to have bugs during maintenance. Test methods and classes used heavily by different parts of program.
2. Run the JUnit test as part of build process
    - Integrate **JUnit** with *build script* so every compile runs tests automatically. *Maven* and *ANT* are popular for Java applications. Isn't just best practice but standard of building Java applications.
3. Always test for boundary conditions
    - Develop test cases based on usage or boundary conditions.
4. Align test with business requirements
5. Test for non-functional requirements
    - i.e. if writing *thread-safe class* try and write tests to break thread-safety
6. Test for ordering
    - If function depends on order of events.
7. Use *@Ignore* annotation
    - Create dummy tests when working with requirements, best time to remember requirements and you can add a comment describing the intent.
8. Avoid testing simple getter methods
9. Avoid dependence on database and file system
    - Keep the unit test independent of environmental data, as if this changes it'll break the tests.
10. Use tools


## JUnit Annotations and Assertions
Parameter order is the *expected* value followed by the *actual* value. 
Optionally, the first/last parameter can be a string messsage that is output on failure.

[[ JUnit annotations | https://www.guru99.com/junit-annotations-api.html]]

An annotation is a special form of syntactic meta-data that can be added to Java source code for better code readability and structure. 
Variables, parameters, packages, methods, and classes can be annotated.
All have the prefix of the '@' symbol.
- `Test` : shows the public void method its attached to can be executed as test case
- `Before` : if you want to execute some statement like preconditions before each test case
- `BeforeClass` : execute some statements before *all* test cases
- `After` : execute some statements after each test case i.e. *resetting variables, deleting temp files, variables, etc.*
- `AfterClass` : execute some statement after all test cases i.e. *releasing resources*
- `Ignores` : if you want to ignore some statements during execution, like disabling some test cases
- `Test(timeout=500)` : set a timeout for execution
- `Test(expected=IllegalArgumentException.class)` : handle some exception during execution, like if you're checking a method throws specified exception or not

Assertions examples:
```Java
import static org.hamcrest.CoreMatchers.allOf;
import static org.hamcrest.CoreMatchers.anyOf;
import static org.hamcrest.CoreMatchers.both;
import static org.hamcrest.CoreMatchers.containsString;
import static org.hamcrest.CoreMatchers.equalTo;
import static org.hamcrest.CoreMatchers.everyItem;
import static org.hamcrest.CoreMatchers.hasItems;
import static org.hamcrest.CoreMatchers.not;
import static org.hamcrest.CoreMatchers.sameInstance;
import static org.hamcrest.CoreMatchers.startsWith;
import static org.junit.Assert.assertArrayEquals;
import static org.junit.Assert.assertEquals;
import static org.junit.Assert.assertFalse;
import static org.junit.Assert.assertNotNull;
import static org.junit.Assert.assertNotSame;
import static org.junit.Assert.assertNull;
import static org.junit.Assert.assertSame;
import static org.junit.Assert.assertThat;
import static org.junit.Assert.assertTrue;

import java.util.Arrays;

import org.hamcrest.core.CombinableMatcher;
import org.junit.Test;

public class AssertTests {
  @Test
  public void testAssertArrayEquals() {
    byte[] expected = "trial".getBytes();
    byte[] actual = "trial".getBytes();
    assertArrayEquals("failure - byte arrays not same", expected, actual);
  }

  @Test
  public void testAssertEquals() {
    assertEquals("failure - strings are not equal", "text", "text");
  }

  @Test
  public void testAssertFalse() {
    assertFalse("failure - should be false", false);
  }

  @Test
  public void testAssertNotNull() {
    assertNotNull("should not be null", new Object());
  }

  @Test
  public void testAssertNotSame() {
    assertNotSame("should not be same Object", new Object(), new Object());
  }

  @Test
  public void testAssertNull() {
    assertNull("should be null", null);
  }

  @Test
  public void testAssertSame() {
    Integer aNumber = Integer.valueOf(768);
    assertSame("should be same", aNumber, aNumber);
  }

  // JUnit Matchers assertThat
  @Test
  public void testAssertThatBothContainsString() {
    assertThat("albumen", both(containsString("a")).and(containsString("b")));
  }

  @Test
  public void testAssertThatHasItems() {
    assertThat(Arrays.asList("one", "two", "three"), hasItems("one", "three"));
  }

  @Test
  public void testAssertThatEveryItemContainsString() {
    assertThat(Arrays.asList(new String[] { "fun", "ban", "net" }), everyItem(containsString("n")));
  }

  // Core Hamcrest Matchers with assertThat
  @Test
  public void testAssertThatHamcrestCoreMatchers() {
    assertThat("good", allOf(equalTo("good"), startsWith("good")));
    assertThat("good", not(allOf(equalTo("bad"), equalTo("good"))));
    assertThat("good", anyOf(equalTo("bad"), equalTo("good")));
    assertThat(7, not(CombinableMatcher.<Integer> either(equalTo(3)).or(equalTo(4))));
    assertThat(new Object(), not(sameInstance(new Object())));
  }

  @Test
  public void testAssertTrue() {
    assertTrue("failure - should be true", true);
  }
}
```

## MoreUnit
[[ MoreUnit for Intellij | https://github.com/MoreUnit/org.moreunit.intellij.plugin]]


---
> Useful links:
- [[Best practices | https://javarevisited.blogspot.com/2012/08/best-practices-to-write-junit-test.html#axzz7kd6JcAfW]]