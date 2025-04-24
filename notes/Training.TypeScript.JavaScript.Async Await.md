---
id: y80p5x6r8awi2lhflobjhjl
title: Async Await
desc: 'Surrounding promises and asynchronous functions that get called with Await'
updated: 1745425507942
created: 1745418707975
---
## Using Promises
[MDN Using Promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises)<br>
A `Promise` is an object representing eventual completion or failure of an asynchronous operation, essentially its an object that you attach callbacks to instead of passing callbacks into a function.
> [Callbacks](https://www.w3schools.com/js/js_callback.asp) are *functions passed as an argument to another function*

```Js
// Create an Array
const myNumbers = [4, 1, -20, -7, 5, 9, -6];

// Call removeNeg with a callback
const posNumbers = removeNeg(myNumbers, (x) => x >= 0);

// Display Result
document.getElementById("demo").innerHTML = posNumbers;

// Keep only positive numbers
function removeNeg(numbers, callback) {
  const myArray = [];
  for (const x of numbers) {
    if (callback(x)) {
      myArray.push(x);
    }
  }
  return myArray;
}
```
Above `(x) => x >= 0` is a **callback function**, being passed to `removeNeg()` as an argument.

Callbacks really shine when using *asynchronous functions* where one function must wait for another function like waiting for a file to load.

## Promises and Callback Hell
A common need is to execute 2+ async operations back to back, and each subsequent operation starts when the last succeeds. Callback hell is effectively when you have lots of methods chained together at once or one big function that inside it has lots of nested functions, see [Callback Hell](http://callbackhell.com/).

```Js
doSomething(function (result) {
  doSomethingElse(result, function (newResult) {
    doThirdThing(newResult, function (finalResult) {
      console.log(`Got the final result: ${finalResult}`);
    }, failureCallback);
  }, failureCallback);
}, failureCallback);
```
We can create a promise chain to accomplish this, the API design of promises makes it easy because callbacks are attached to the returned promise object.
Below, the `then()` function returns a **new promise**, different from the original.
```Js
const promise = doSomething();
const promise2 = promise.then(successCallback, failureCallback);
```
Could do something along these lines
```Js
doSomething()
  .then((result) => doSomethingElse(result))
  .then((newResult) => doThirdThing(newResult))
  .then((finalResult) => {
    console.log(`Got the final result: ${finalResult}`);
  })
  .catch(failureCallback);
```
However this would return the same error for any failure.
> Note: Arrow function expressions can have an *implicit return* so `() => x` is short for `() => { return x; }`

You **must** return promises from `then` callbacks, even if the promise resolves to `undefined`, if you don't then you can't track its settlement and the promise is "**floating**".
```Js
doSomething()
  .then((url) => {
    // Missing `return` keyword in front of fetch(url).
    fetch(url);
  })
  .then((result) => {
    // result is undefined, because nothing is returned from the previous
    // handler. There's no way to know the return value of the fetch()
    // call anymore, or whether it succeeded at all.
  });
  // =================== Won't work properly =====================
  doSomething()
  .then((url) => {
    // `return` keyword added
    return fetch(url);
  })
  .then((result) => {
    // result is a Response object
  });
```
There are four composition tools for running aynchronous operations concurrently: `Promise.all(), Promise.allSettled(), Promise.any(), Promise.race()`
- Fulfills when **all** of the promises fulfill; rejects when **any** reject
- Fulfills when **all** promises settle
- Fulfills when **any** of the promises fulfill, reject when **all** reject
- Settles when **any** of the promises settle, fulfills when any of them fulfills, rejects when any reject

Where possible use await/async logic
```js
async function logIngredients() {
  const url = await doSomething();
  const res = await fetch(url);
  const data = await res.json();
  listOfIngredients.push(data);
  console.log(listOfIngredients);
}
```

### Converting promise chains to async await
```js
async function fetchProducts() {
  try {
    // after this line, our function will wait for the `fetch()` call to be settled
    // the `fetch()` call will either return a Response or throw an error
    const response = await fetch(
      "https://mdn.github.io/learning-area/javascript/apis/fetching-data/can-store/products.json",
    );
    if (!response.ok) {
      throw new Error(`HTTP error: ${response.status}`);
    }
    // after this line, our function will wait for the `response.json()` call to be settled
    // the `response.json()` call will either return the parsed JSON object or throw an error
    const data = await response.json();
    console.log(data[0].name);
  } catch (error) {
    console.error(`Could not get products: ${error}`);
  }
}

fetchProducts();
```
Is seen as better and cleaner than
```js
const fetchPromise = fetch(
  "https://mdn.github.io/learning-area/javascript/apis/fetching-data/can-store/products.json",
);

fetchPromise
  .then((response) => {
    if (!response.ok) {
      throw new Error(`HTTP error: ${response.status}`);
    }
    return response.json();
  })
  .then((data) => {
    console.log(data[0].name);
  });
```
But the next is even better as we're moving the try and catch back to the catch handler on the returned promise.
```js
async function fetchProducts() {
  const response = await fetch(
    "https://mdn.github.io/learning-area/javascript/apis/fetching-data/can-store/products.json",
  );
  if (!response.ok) {
    throw new Error(`HTTP error: ${response.status}`);
  }
  const data = await response.json();
  return data;
}

const promise = fetchProducts();
promise
  .then((data) => {
    console.log(data[0].name);
  })
  .catch((error) => {
    console.error(`Could not get products: ${error}`);
  });
```


> ### Summary
> - Dont' nest functions, give them names and place them at the top level
> - Use function hoisting to your advantage to move functions 'below the fold'
> - Handle **every single error** in every one of your callbacks, use a linter to help with it
> - Create reusable functions and place them in a module to reduce cognitive load required to understand code, splitting code into small pieces also helps error handling and test writing, it forces creation of a stable and documented public API for your code.

