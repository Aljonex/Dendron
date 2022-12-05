---
id: 2rgyftzf1epf699xi09wibl
title: Tut1-5
desc: ''
updated: 1670263766430
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
- [[Training.Intellij.Modifiers]] 

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

## Learned from PR
- Composition: A means of avoiding inheritance, involves a constructor instantiating the object that would have been extended. Make sure to make use of polymorphism i.e. if object you wanted was implementing an interface, use that interface to future proof for extra extensions.
- **Static** : variables always at the top
- In testing classes you don't have to prefix names with test as it is assumed they are being tested therefore you can name them like `private <ClassName> className`
- When using *assertJ* or assertions in testing you should use the string formatting of `%s` for example instead of `{}`
- The logger can make use of `{}`
- Mocking is for mocking a dependency of a class, stubbing is to fake data being returned
- [[JUnitParamsRunner.class | https://github.com/Pragmatists/JUnitParams]] is the page for parameterised testing, you are able to mix parameterised with non-parameterised which is different from standard runner [[some example | https://github.com/Pragmatists/JUnitParams/blob/master/src/test/java/junitparams/usage/SamplesOfUsageTest.java]]
    - Generally speaking [[when using JUnitParams | https://www.baeldung.com/junit-params]] you have the `@Parameters` notation underneath `@Test` which will automatically search for a mapping of `paramtersFor<testMethodName>` if this isn't how you've named it then you can add `@Parameters(method="methodName")`
- Variables should have new line between them
- Test name is `<testMethodName_whenCertainCondition>`
- Setup and teardown should be at the top of a test file to be clearer
- To remove files that are tracked but in the .gitignore use `git rm --cached <filename>`
- When using assertions in testing the `.as("desc")` should always be before the comparison, and with big decimal use `.isCloseTo` instead of `isEqualTo`
- `NullPointerExceptions` always are runtime exceptions, you should use `IllegalArgumentException` instead which is also a runtime exception but is better
- With logger, mainly use `LOGGER.debug` instead of `LOGGER.info`
- When using composition be wise to make use of polymorphism which is **the ability of a class to be *used* as different classes/interfaces**, i.e. 
```Java
public class CAPExtraValueCalculator implements ValuationCalculator {

    private static final Logger LOGGER = LoggerFactory.getLogger(CAPExtraValueCalculator.class);

    private final ValuationCalculator calculator;

    public CAPExtraValueCalculator(ValuationCalculator valuationCalculator) {
        this.calculator = valuationCalculator;
    }
    ...
}

public class CAPExtraValueCalculatorTest {
    private CAPValuationCalculator mockValuationCalculator;

    private CAPExtraValueCalculator spyCapExtension;

    @Before
    public void setup() {
        mockValuationCalculator = Mockito.mock(CAPValuationCalculator.class);
        spyCapExtension = Mockito.spy(new CAPExtraValueCalculator(mockValuationCalculator));
    }
    ...
}
```
- @BeforeClass and @AfterClass annotations are for heavier operations like database setup and teardown, not for variable setting
- No `this.` annotation in getters

Some useful imports for testing:
```Java
import static org.assertj.core.api.AssertionsForClassTypes.within;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.ArgumentMatchers.anyInt;
import static org.assertj.core.api.Assertions.assertThatThrownBy;
import static org.mockito.Mockito.times;
import static org.mockito.Mockito.verify;
import static org.mockito.Mockito.when;

import org.assertj.core.api.JUnitSoftAssertions;
import org.junit.Before;
import org.junit.Rule;
import org.junit.Test;
import org.junit.runner.RunWith;

import org.mockito.Mockito;
```