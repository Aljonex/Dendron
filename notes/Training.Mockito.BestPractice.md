---
id: jqqvnassvltymgaxgbkagqw
title: BestPractice
desc: ''
updated: 1669212926121
created: 1669206147657
---
Discouraged ugly imports
```Java
import org.mockito.Mockito;
import org.mockito.Matchers;
 
@Test
public void testSomething() {
    MyClass mcl = Mockito.mock(MyClass.class);
    Mockito.when(mcl.myMethod(Matchers.eq(true), Matchers.any(OtherClass.class)));
}
```
Encouraged static imports
```Java
import static org.mockito.Mockito.mock;
import static org.mockito.Mockito.when;
import static org.mockito.Matchers.eq;
import static org.mockito.Matchers.any;
 
@Test
public void testSomething() {
    MyClass mcl = mock(MyClass.class);
    when(mcl.myMethod(eq(true), any(OtherClass.class)));
}
```

### Model Objects
Avoid using mocks of the model objects.
They don't have behaviour associated with them, so it's just as easy to just make some model objects and use those.