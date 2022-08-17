---
title: "SSG and SSR in Next project"
date: 2022-08-02T00:05:06-04:00
author: "ozan ocak"
tags: ["NextJs", "web development"]
---

## Statically Generated Pages in Server

**getStaticProps** method must return an object created in build time in server, since the page is statically created it will be very fast loaded. We can see loading in Network/ waterfall.

```javascript
export const getStaticProps = () => {
  return {
    props: {
      name: "ozan",
    },
  };
};
const Post = ({ name }) => {
  return (
    <div>
      <h1>Post by {name}</h1>
    </div>
  );
};
export default Post;
```

We can fetch data from API and create the page in server. When the page is loaded in browser , it will be plain HTML because javascript run in the backend so loaded time will be faster.

```javascript
export const getStaticProps = async () => {
  const post = await fetch("https://jsonplaceholder.typicode.com/posts/5");
  const data = await post.json();
  return {
    props: {
      post: data || null,
    },
  };
};
const Post = ({ post }) => {
  return (
    <div>
      <h1>{post.title}</h1>
      <p>{post.body}</p>
    </div>
  );
};
export default Post;
```

when we search **'/post'** in the url to get statically fetch data

## Dynamically created pages

we can dynamically create pages under **'/post/[postId].js'** , but we have to either predefine /post/[postId] and pass it as an argument to getStaticProps which is still statically rendered in build time; below code _/posts/5_ and _/posts/10_ are statically rendered

```javascript
export const getStaticPaths = () => {
  return {
    paths: [{ params: { postId: "5" } }, { params: { postId: "10" } }],
    fallback: false,
  };
};

export const getStaticProps = async ({ params }) => {
  const data = await fetch(
    `https://jsonplaceholder.typicode.com/posts/${params}`
  );
  const serializedData = await data.json();
  return {
    props: {
      post: serializedData || null,
    },
  };
};
```

and finally we can dynamically produced pages in client side we need return _fallback: true_ from getStaticPaths method to do that and check fallback to render the data we get as an arguments from getStaticProps method.

```javascript
const PostDetails = ({ post }) => {
  const router = useRouter();
  //const { postId } = router.query;
  //console.log(router);

  return router.isFallback ? (
    <h1>loading....</h1>
  ) : (
    <div>
      <h1>{post.title}</h1>
      <p>{post.body}</p>
    </div>
  );
};
export default PostDetails;
```

## Server Side Rendering

Server Side rendered pages build in run time in the server and later fetched by _getServerSideProps_ method.

```javascript
export const getServerSideProps = async ({ query }) => {
  const usr = await fetch(
    `https://jsonplaceholder.typicode.com/users/${query.id}`
  );
  const userSerialized = await usr.json();
  return {
    props: { user: userSerialized || null },
  };
};

const Profile = ({ user }) => {
  return !Object.keys(user).length ? (
    <div>Invalid User Id</div>
  ) : (
    <div>
      <div>{user.name}</div>
      <div>{user.email}</div>
    </div>
  );
};
export default Profile;
```
