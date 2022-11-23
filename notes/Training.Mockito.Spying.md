---
id: nod1wv24p6j9s09lhntg63n
title: Spying
desc: ''
updated: 1669206096537
created: 1669205197488
---
We can do more than create dummy objects too.
You can 'spy' on the interaction of real object, this is considered a "**code smell**" or something that links to bad design.
If you have to spy on an object it indicates you can't mock it, or you're having to test how many times a real object is called.
Even though its discouraged, here's how you do it.
```Java
AgreementService realAgreementService = getAgreementService();
AgreementService gchq = spy(realAgreementService);
BrandNewProcess np = new BrandNewProcess();
np.setAgreementService(gchq);
 
np.performSomeActionThatDoesntReturnAnything(asset);
 
verify(gchq).amend(eq(asset.getAgreementInvoice()), eq(asset), eq(128), eq(true));
```
Here, the AgreementService is real with a spy object wrapped around it.
Objects can interact with the spy as if its normal and it'll redirect its execution to the real object in all cases.
Then later we can verify the number of times certain methods were called, exactly like we did with our mock earlier.

**The advantage** is if there's complex, highly coupled behaviour between AgreementService and BrandNewProcess then we don't have to try and mock it, lets us view whats going on and check everything is as expected.