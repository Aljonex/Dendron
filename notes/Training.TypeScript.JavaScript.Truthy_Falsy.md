---
id: yowxnhlzja2p7vh22lqcefh
title: Truthy Falsy
desc: ''
updated: 1745418700232
created: 1745415528635
---
# Truthy and Falsy
Comparison of 2 things can trip you up in JavaScript.
- JavaScript variables are *loosely typed*, which allows flexible assignments. JavaScript converts values to string representation before comparison with loose equality operator (**==**) but using *strict equality* (**===**) considers the type, which allows for more predictability.
- Every value in JS has an inherent Boolean value, known as either **Truthy or Falsy**, Falsy includes `false, 0, -0n, "", null, undefined, NaN` whereas everything else is truthy including `'0', 'false', [], {}, function()[]` this is paramount for good debugging

```Javascript
let x;
x = 1;   // x is a number
x = '1'; // x is a string
x = [1]; // x is an array

// all true
1 == '1';
1 == [1];
'1' == [1];
```

Moral of the story is its probably best to use the strict equality, as it restrains it more.

[Truthy Falsy guide](https://www.sitepoint.com/javascript-truthy-falsy/)