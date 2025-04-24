---
id: zvj73nd6mmxdp295h3wdrsc
title: Interfaces
desc: ''
updated: 1745508043408
created: 1745506904819
---
[Interfaces in the handbook](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#interfaces)

```ts
interface Point {
  x: number;
  y: number;
}
 
function printCoord(pt: Point) {
  console.log("The coordinate's x value is " + pt.x);
  console.log("The coordinate's y value is " + pt.y);
}
 
printCoord({ x: 100, y: 100 });
```
Just another way to name an object type.
```ts
interface Animal {
  name: string;
}

interface Bear extends Animal {
  honey: boolean;
}

const bear = getBear();
bear.name;
bear.honey;
        

type Animal = {
  name: string;
}

type Bear = Animal & { 
  honey: boolean;
}

const bear = getBear();
bear.name;
bear.honey;
```
Only types can have primitives (think `null, undefinde, 'stringLiteral', and also unions of types.)

You can *combine literal* into unions, and express functions only accepting certain set of known values.
```ts
function printText(s: string, alignment: "left" | "right" | "center") {
  // ...
}
printText("Hello, world", "left");
printText("G'day, mate", "centre");
```