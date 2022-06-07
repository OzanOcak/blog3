---
title: "Memory Management in JavaScript and Necessity of Spread Operator in React"
date: 2022-06-06T00:05:06-04:00
author: "ozan ocak"
tags: ["react", "js", "memory management"]
---

## Memory Management in JavaScript and Necessity of Spread Operator in React

Descartes believed dualism of mind and body,he thought they couldn't have the same origin since body is divisible,while mind is not; only one difference was enough to make two different objects according to him.

Let's think the two identical pencils are produced in the factory, everything but their positions in the world is matched so they are not exactly same. if everything would be same , they would come together and become one single object or one pencil.

```js
[1, 2, 3] === [1, 2, 3]
  ? console.log("they are same")
  : console.log("they are not same");
```

Above code snippet is exact replicate of two pencils thought. Javascript Engine located in the browsers is a compiler of the javascript language that handle memory management and garbage collection. Since javascript is dynamically typed language, when above code
is executed, the engine dynamically allocate two same arrays in different locations and eventually the code will return **they are not same**.

Javascript has some problems because of the design of the language and even typescript doesn't catch them. Although their values are matched, the different address location makes them two different objects.

Everything beside primitive data types is objects in javascript and objects are stored in heap therefore they pass by reference.There is one catch in javascript; null is also stored in heap, it is forgotten to assign pointer so it is also an object.

I recently learned react.js and the react library relies on creating a new states in every steps of app. Then they compare to values of the states, if there is change, they will update and render the virtual virtual dom .

The problem is if we pass an array or an object which is also dynamically allocated in heap by the engine, react library will not recognize any change because they will pass by reference since they are allocated in the heap.

```js
try {
      const response = await api.put(`/posts/${id}`, updatedPost);
      setPosts(
        posts.map((post) => (post.id === id ? { ...response.data } : post))
      );
```

This is an async updating data operation; the updated data is assigned to response.
in the second line we map all the posts one by one to find the updated post's id, if we find id we can update it, but as mentioned, we are passing exact data itself by reference, response is just a tag. that's where spread operator comes to play and duplicate the state now react can render the dom.

It is like a computer and electricity that gives mind to the computer. While computer is made of transistors, metals,cables and plastics. Electricity is moving sub atomic particles is assumably exist, forming the energy around and giving it back later. Descartes might be right that body and mind could be dual...
