---
title: "How to Install tailwindcss and typescript in Next project"
date: 2022-08-01T00:05:06-04:00
author: "ozan ocak"
tags: ["NextJs", "web development", "tailwindcss", "typescript"]
---

## How to create Next Js project with Typescript and tailwindcss

We can create next app with yarn include --typescript in the end if we want tsx files.

```console
yarn create next-app rentalapp --typescript
cd rentalapp
yarn add -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

by executing npx tailwind creates 2 config files

tailwind.config.js

```javascript
module.exports = {
  content: [
    "./pages/**/*.{js,ts,jsx,tsx}",
    "./components/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
};
```

and finally we should insert bellow snippet into globals.css

styles/globals.css

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

```console
yarn dev
```
