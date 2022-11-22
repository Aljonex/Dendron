---
id: 17rg7xlxcuo0n1iztgoejlo
title: JUnit
desc: ''
updated: 1668507822040
created: 1668426100214
---
> Learning Objectives
- Different kinds of test: *Unit, Integration, End to End*
- Differences between automated and manual testing
- Testing best practices
- JUnit Annotations and Assertions
- MoreUnit
- How to create a test class
- How to test client Configuration
- How to calculate Code Coverage and why it is useful

In Sopra we use JUnit 4 which on the maven repo is still under the groupId and artifactId `junit` (in the pom) but JUnit5 is under `org.junit.jupiter` and `junit-jupiter-api` for groupId and artifactId respectively.

We use *AssertJ* over *Hamcrest*, from the internet it seems that AssertJ is more popular for the single API entrypoint. Where JUnit is a testing framework, *AssertJ* has much deeper assertions and provides much more readability.

*AssertJ* is a library that provides a rich set of assertions, truly helpful error messages; its designed with ease of use in mind and has easy assertions to use. You can type `.assertThat` followed by the actual value in parentheses and a dot and the IDE will show you assertions available.
```Java
import static org.assertj.core.api.Assertions.assertThat;  // main one
import static org.assertj.core.api.Assertions.atIndex; // for List assertions
import static org.assertj.core.api.Assertions.entry;  // for Map assertions
import static org.assertj.core.api.Assertions.tuple; // when extracting several properties at once
import static org.assertj.core.api.Assertions.fail; // use when writing exception tests
import static org.assertj.core.api.Assertions.failBecauseExceptionWasNotThrown; // idem
import static org.assertj.core.api.Assertions.filter; // for Iterable/Array assertions
import static org.assertj.core.api.Assertions.offset; // for floating number assertions
import static org.assertj.core.api.Assertions.anyOf; // use with Condition
import static org.assertj.core.api.Assertions.contentOf; // use with File assertions
```

## Creating Test class
[[ Vogella - JUnit 5 example | https://www.vogella.com/tutorials/JUnit/article.html]]

A JUnit test is a method contained in a class just used for testing.
Convention is the *name of the class* followed by Test i.e. `CAPValuationCalculatorTest`.
See [[Training.JUnit.JUnitBasics]] for examples.

## Test client configuration

## Calculate code coverage 

### Useful Resources
- [[Getting Started | https://github.com/junit-team/junit/wiki/Getting-started]]
- [[ Assertions | https://github.com/junit-team/junit/wiki/Assertions]]
- [[ AssertJ | http://joel-costigliola.github.io/assertj/index.html]]
- [[ WFS wiki: Testing | https://confluence.apak.com/live/display/WIKI/Testing+WFS]]
- [[ WFS wiki: JUnit | https://confluence.apak.com/live/display/WIKI/JUnit]]
- [[ JUnit4 to 5 | https://www.baeldung.com/junit-5-migration]]