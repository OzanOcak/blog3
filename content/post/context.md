---
title: "React Hooks in Next.js"
date: 2022-08-16T00:05:06-04:00
author: "ozan ocak"
tags: ["js", "next.js", "state management", "react hooks"]
---

## State Management using react hooks in Next.js

First we nee create a store then export it with _useContext_ hook

```javascript
import { createContext, useContext } from "react";

const Store = createContext();

export const useStore = () => useContext(Store);
```

Then we need create a reducer function takes state and action as an arguments, we will
change the state base on action type with switch

```javascript
const reducer = (state, action) => {
  console.log(action);

  switch (action.type) {
    default: {
      return state;
    }
  }
};
```

A store provider needs to be created and the reducer with actions and states need to be assign to it; action will be dispatched.

```javascript
export const StoreProvider = ({ children }) => {
  const [state, dispatch] = useReducer(reducer, {});

  return <Store.Provider value={[state, dispatch]}>{children}</Store.Provider>;
};
```

we can provide dummy data via useReducer hook.

```javascript
const [state, dispatch] = useReducer(reducer, { name: "donald duck" });
```

if we wrap <Layout> with **<StoreProvider>** in \_app.js file, we can access state in anywhere in the app.

we can call usStore within header.jsx and reach to state.

```javascript
const [state, dispatch] = useStore();
console.log({ state });
```

### Constants

```javascript
export const authConstants = {
  LOGIN_REQUEST: "LOGIN_REQUEST",
  LOGIN_SUCCESS: "LOGIN_SUCCESS",
  LOGIN_FAILURE: "LOGIN_FAILURE",
};
```

now we can place real use authentication data in useReducer instead of name:{}

```javascript

user:{
    authenticated:false,
    authenticating:false,
    error:null
}
```

we can dispatch type of the action and initiate state change in login.jsx

````javascript
const payload = { email, password };
    dispatch({ type: authConstants.LOGIN_REQUEST });
    const res = await signIn("credentials", { ...payload, redirect: false }); //next-auth
    console.log({ res });
    if (!res.error) {
      const session = await getSession();
      dispatch({ type: authConstants.LOGIN_SUCCESS, payload: session });
      router.replace("/");
    } else {
      dispatch({ type: authConstants.LOGIN_FAILURE, payload: res.error });
      setErrorMessage(res.error);
    }
    ```
````

if we define all the action types in context/index.js , we can dispatch them and our state management will be completed

```javascript
switch (action.type) {
  case authConstants.LOGIN_REQUEST: {
    return { ...state, user: { authenticating: true, ...state.user } };
  }
  case authConstants.LOGIN_SUCCESS: {
    return {
      ...state,
      user: {
        authenticating: false,
        authenticated: true,
        ...action.payload.user,
      },
    };
  }
  case authConstants.LOGIN_FAILURE: {
    return {
      ...state,
      user: {
        ...state.user,
        error: action.payload,
      },
    };
  }
  default: {
    return state;
  }
}
```

Finally we can fetch state in the client side end update interface accordingly

```javascript
const [state, dispatch] = useStore();
const user = getValue(state, ["user"]); // state.user
const authenticated = getValue(state, ["user", "authenticated"], false);
// state.user.authenticated
{authenticated ? (
          <button>log out</button>
        ) : ( <button>log in</button> }
```

### state management in sign up pages

```javascript
const [state] = useStore();
const user = getValue(state, ["user"], null);
// look at state.user, if it empty assign null
```
