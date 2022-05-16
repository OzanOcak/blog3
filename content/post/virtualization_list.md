---
title: "Virtualization creating list"
date: 2022-05-16T00:05:06-04:00
author: "ozan ocak"
tags: ["javascript", "web development", "DOM", "css", "html"]
---

## Virtualization creating list

Firstly we need to crate index.html where we should link css and javascript files

```html
<html lang="en">
  <head>
    <link rel="stylesheet" href="style.css" />
  </head>
  <body>
    <script src="./main.js"></script>
  </body>
</html>
```

Then we need a text input to receive the texts,
a button to bind function to pass the text to dom by creating html elements
an unordered list where we can insert list element in.

```html
<input type="text" id="input" />
<button id="btn">add</button>
<div id="list-container">
  <ul id="list"></ul>
</div>
```

Now we can get button element to give it functionality within main.js

```js
let button = document.getElementById("btn");
```

when we click on button an anonymous function will be assigned to onclick method, so this is writing implementation for onclick method

```js
button.onclick = function () {
```

we are going to get input element and assign the value is entered to variable
then check if it is null.

```js
const input = document.getElementById("input").value;
if (!input) return null;
```

we are going to create an html element, a child node and append it to html element.
html element is all the html tags while html nodes all the child nodes with in tag.

```js
const node = document.createElement("li");
const text_node = document.createTextNode(input);
node.appendChild(text_node);
```

we will get list which is ul tag and will append list node to it

```js
let list = document.getElementById("list");
list.appendChild(node);
```

Lastly we will add a class name to style via css, every 3.rd child of list element a class will be added.
the important point here; to reach child elements we can use **children** while to reach child nodes we use **childNode** methods.

```js
list.children.length % 3 === 0 ? (node.className += "red") : null;
```

style.css

```css
.red {
  color: red;
}
```
