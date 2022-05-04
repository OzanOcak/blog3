---
title: "The language with two null pointers"
date: 2022-03-31T00:05:06-04:00
author: "ozan ocak"
tags: ["js", "website"]
---

### Javascript is a beautiful language

As a sharepoint front-end developer, I use javascript and jquery for last 6 years. I never tried to understand it because of the backlash toward the language. Apparently it was a fault since I think there us nothing wrong with it.

Almost all the languages are not secure if you don't give enough attention to code. For sure JavaScript requires little bit more.

### What are the problems and their solutions

- Avoid Global Variables. Minimize the use of global variables because the are located in the heap with windows scope which means they are not garbage collected.

- Always Declare Local Variables so garbage collector will unlink the data when the program is out of scope.

- Declarations on Top because hoisting is confusing, if the variable is hoisted it is referenced to null pointer in the memory, and it is assigned undefined which is object ( undefined is a type unlike null. )

- Initialize Variables with the same reason

- Declare Objects and Arrays with const because const and let are not store in global scope, the are stored in script scope.

- Don't use new object for String, Number and Boolean

- Be caution in type conversion

- Use triple equal sign

- Avoid eval

### Best Practice in JavaScript

```javascript
let l = arr.length;
for (let i = 0; i < l; i++) {
```

- Reduce Dom access with innerHTML

- use getElementsByTagName selector for better performance

- do not create new dom variable to assign it, use one liner instead

```javascript
var fragment = document.createDocumentFragment();
```

- to create new dom nodes use createDocumentFragment for better performance
