---
id: kf2u29g5c7i9r3nns694wp4
title: Intro
desc: ''
updated: 1678457824321
created: 1678450812250
---
JavaScript or JS is inextricably linked to html as its language `.js` files, are *scripts* which can be written directly into a webpage's HTML or imported and run automatically when the page loads.

A basic example of an `index.html` file:
```html
<!DOCTYPE html>
<html>
    <head>
        <title>Learning</title>
    </head>
    <body>
        <script src="helloworld.js"></script>
        <script>alert("Hello world!")</script>

        <p id="testsection"></p>

        <script>
            document.getElementById("testsection").innerHTML = "First js program!"
        </script>
    </body>
</html>
```
Shown above is that scripts can be made and run directly in an html file or can be imported from a source file.

`"use strict";` being set at the top of a script or function defines that old code should be made to work in the following section.
There are some *classes* and *modules* that enable `use strict` automatically so that the directive doesn't need adding.

## Variables
A variable is *named storage* for data, and you can store loads of things, to declare you must use the `let` keyword.
Older scripts have a `var` keyword instead, these variables cannot start with a digit or have hyphens, but can start with `$` or `_`.