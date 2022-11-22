---
id: afunq4jgfn26w0dzv5y3gh6
title: BigDecimal
desc: ''
updated: 1668425867710
created: 1668423658223
---
Use of BigDecimal over *int* or *long*. An exact representation of decimal numbers, has **no comparison** errors and **no rounding** errors so its great for representing currency, it is immutable (which is a pro) but means calculations become verbose, so we use the [[string constructor | https://docs.oracle.com/javase/7/docs/api/java/math/BigDecimal.html#BigDecimal(java.lang.String)]].

Computers generally use a format called *binary floating-point* which can't accurately represent numbers like 0.1, 0.2, 0.3. 
When the code is compiled, the number is rounded to the nearest number in that format, which results in small rounding errors before the calculation.
The reason this is used because its much faster and for most calculations a small error in the *n<sup>th</sup>* place usually won't matter.

> Useful Links:
- [[Big Decimal Oracle | https://docs.oracle.com/javase/7/docs/api/java/math/BigDecimal.html]]