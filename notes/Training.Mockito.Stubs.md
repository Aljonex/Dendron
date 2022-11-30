---
id: 8x15zm3omxfidmriozan7rd
title: Stubs
desc: ''
updated: 1669822333927
created: 1669375226474
---

### Mock
A class that implements an interface and allows ability to dynamically set values to *return/exceptions* to throw from particular methods and provides the ability to check if particular methods have been called/not called.
You can check what you're expecting to receive.

### Stub
Like a mock class, but it doesn't provide the ability to verify methods have been called/not called. 
These won't throw an error and will return a pre-defined value.

```Java
 @Spy private ValuationCalculator capSpy = new CAPExtension(valuationCalculator);

 doReturn(new BigDecimal("value")).when(capSpy).methodName(var1, var2);
```

### Fake
A class that implements an interface but contains fixed data and no logic. 
Simply returns "good" or "bad" data depending on the implementation.

Can use Mock when its an object that returns a value that is set to tested class.
Use stub to mimic an *Interface or Abstract class* to be tested, they're classes not used in production basically.