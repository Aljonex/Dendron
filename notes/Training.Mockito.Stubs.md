---
id: 8x15zm3omxfidmriozan7rd
title: Stubs
desc: ''
updated: 1669376167069
created: 1669375226474
---

### Mock
A class that implements an interface and allows ability to dynamically set values to *return/exceptions* to throw from particular methods and provides the ability to check if particular methods have been called/not called.

### Stub
Like a mock class, but it doesn't provide the ability to verify methods have been called/not called.

### Fake
A class that implements an interface but contains fixed data and no logic. 
Simply returns "good" or "bad" data depending on the implementation.

Can use Mock when its an object that returns a value that is set to tested class.
Use stub to mimic an *Interface or Abstract class* to be tested, they're classes not used in production basically.