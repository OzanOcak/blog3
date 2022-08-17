---
title: "Creating API within Next.js"
date: 2022-08-05T00:05:06-04:00
author: "ozan ocak"
tags: ["NextJs", "web development"]
---

## Creating API within Next.js

**mongoose** is used for creating database schema, we create a user model

```javascript
import mongoose from "mongoose";

const UserSchema = mongoose.Schema(
  {
    name: { type: String, required: true, trim: true },
    email: { type: String, required: true, unique: true, index: true },
    password: { type: String, required: true, trim: true },
  },
  { timestamps: true }
);

export default mongoose.model("User", UserSchema);
```

creating utility file and defining all the necessary methods to handle the api response is the clean code we should follow.

```javascript
export const errorHandler = (data, res, code = 400) => {
  res.status(code).json({
    hasError: true,
    errorMessage: data,
  });
};

export const responseHandler = (data, res, code = 200) => {
  res.status(code).json({ hasError: false, body: data });
};

export const validateAll = (fields) => {
  for (let key in fields) {
    if (fields[key].trim() === "") throw `${key} is required`;
  }
};
```

and finally we can create our signup folder under pages/api in order to use Next.js indigenous routing

```javascript
import { dbConnect } from "../../../lib/db-connect";
import User from "../../../src/models/user";
import {
  errorHandler,
  responseHandler,
  validateAll,
} from "../../../src/utils/common";

export default async function handler(req, res) {
  if (req.method !== "POST") errorHandler("Invalid request", res);

  try {
    const { name, email, password } = req.body;
    validateAll({ name, email, password });
    // create db connection
    await dbConnect();
    //create new User object and saved it to db
    const user = new User({ name, email, password });
    const savedUser = await user.save();
    if (savedUser) responseHandler(savedUser, res, 201);
    else errorHandler("something went wrong", res);
    //catching error then finally closing connection
  } catch (error) {
    errorHandler(error, res);
  } finally {
    res.end();
  }
}
```
