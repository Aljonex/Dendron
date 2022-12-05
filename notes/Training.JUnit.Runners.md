---
id: gtorrjrc5ac2sb88bvcaz8l
title: Runners
desc: ''
updated: 1670238698002
created: 1669380277630
---
In order to specify a custom runner, use the `@RunWith` annotation.

A JUnit runner is a class that extends JUnit's abstract Runner class and is responsible for running JUnit tests, typically using reflection (modify behaviour of the abstract class).

For example using the `MockitoJUnitRunner.class` gives you access to the @Mock @Spy annotations.

If switching to the `JUnitParamsRunner.class` then you lose access to these annotations, however can still mock through the use of 
```Java
import org.mockito.Mockito;

@RunWith(JUnitParamsRunner.class)
public classTest {
    private CAPValuationCalculator mockValuationCalculator;

    private CAPExtraValueCalculator spyCapExtension;


    @Before
    public void setup() {
        mockValuationCalculator = Mockito.mock(CAPValuationCalculator.class);
        spyCapExtension = Mockito.spy(new CAPExtraValueCalculator(mockValuationCalculator));
    }
}
```