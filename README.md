# Nextjs

this is the nextjs basic tutorial from the website:

https://nextjs.org/learn/foundations/about-nextjs?utm_source=next-site&utm_medium=homepage-cta&utm_campaign=home

netninja explains it really well:

getStaticProps()
getServerSideProps()
getStaticPaths()

https://www.youtube.com/playlist?list=PL4cUxeGkcC9g9gP2onazU5-2M-AzA8eBw

## assets

### unoptimized

```html
<img src="/images/profile.jpg" alt="Your Name" />
```

### optimized

```js
import Image from 'next/image';

const YourComponent = () => (
  <Image
    src='/images/profile.jpg' // Route of the image file
    height={144} // Desired size with correct aspect ratio
    width={144} // Desired size with correct aspect ratio
    alt='Your Name'
  />
);
```

## metadata

- The '<Head />' component used in '\_document' is not the same as next/head. The '<Head />' component used here should only be used for any '<head>' code that is common for all pages. For all other cases, such as '<title>' tags, we recommend using next/head in your pages or components.

- https://nextjs.org/docs/pages/building-your-application/routing/custom-document

```js
import Head from 'next/head';

// jsx
<Head>
  <title>Create Next App</title>
  <link rel='icon' href='/favicon.ico' />
</Head>;
```

## header script

```js
import Script from 'next/script';

export default function FirstPost() {
  return (
    <>
      <Head>
        <title>First Post</title>
      </Head>
      <Script
        src='https://connect.facebook.net/en_US/sdk.js'
        strategy='lazyOnload'
        onLoad={() =>
          console.log(`script loaded correctly, window.FB has been populated`)
        }
      />
      <h1>First Post</h1>
      <h2>
        <Link href='/'>← Back to home</Link>
      </h2>
    </>
  );
}
```

## Layouts

- layout components are shared across pages

## css

- To load global CSS to your application, create a file called "pages/\_app.js"
- You cannot import global CSS anywhere else.
- global styles can only be imported in 'pages/\_app.js'

```js
/ `pages/_app.js`
import '../styles/global.css';    //global styles can ONLY be applied here.

export default function App({ Component, pageProps }) {
  return <Component {...pageProps} />;
}
```

## Pre-rendering

1. static site generation (pre-render - html generated at build time) - static generation 'with' data / static generation 'without' data

- Static Generation (with Data) using getStaticProps() - tells nextjs you have data dependencies...when pre-rendering page at build, deal with them first.
- getStaticProps() only runs on the server-side at BUILD TIME...
- you won’t be able to use data that’s only available during request time, such as query parameters or HTTP headers.
- getStaticProps can only be exported from a page. You can’t export it from non-page files.

```js
export default function Home(props) { ... }

export async function getStaticProps() {
  // Get external data from the file system, API, DB, etc.
  const data = ...

  // The value of the `props` key will be
  //  passed to the `Home` component
  return {
    props: ...
  }
}

```

- Fetch External API or Query Database
  In lib/posts.js, we’ve implemented getSortedPostsData which fetches data from the file system. But you can fetch the data from other sources, like an external API endpoint, and it’ll work just fine:

```js
export async function getSortedPostsData() {
  // Instead of the file system,
  // fetch post data from an external API endpoint
  const res = await fetch('..');
  return res.json();
}
```

### query a db directly

```js
import someDatabaseSDK from 'someDatabaseSDK'

const databaseClient = someDatabaseSDK.createClient(...)

export async function getSortedPostsData() {
  // Instead of the file system,
  // fetch post data from a database
  return databaseClient.query('SELECT posts...')
}
```

2. server side rendering (generates html at each request)

### getServerSideProps()

- https://nextjs.org/docs/basic-features/data-fetching#getserversideprops-server-side-rendering
- To use Server-side Rendering, you need to export getServerSideProps instead of getStaticProps from your page.
- Time to first byte (TTFB) will be slower than getStaticProps because the server must compute the result on every request, and the result cannot be cached by a CDN without extra configuration.

```js
export async function getServerSideProps(context) {
  return {
    props: {
      // props for your component
    },
  };
}
```

3. Client-side Rendering

- Statically generate (pre-render) parts of the page that do not require external data.
  When the page loads, fetch external data from the client using JavaScript and populate the remaining parts.

4. SWR - react hooks for data-fetching

- stale-while-revalidate - SWR is a strategy to first return the data from cache (stale), then send the fetch request (revalidate), and finally come with the up-to-date data.

- https://swr.vercel.app/

```js
import useSWR from 'swr';

function Profile() {
  const { data, error, isLoading } = useSWR('/api/user', fetcher);

  if (error) return <div>failed to load</div>;
  if (isLoading) return <div>loading...</div>;
  return <div>hello {data.name}!</div>;
}
```

## getStaticPaths()

- How to Statically Generate Pages with Dynamic Routes
  In our case, we want to create dynamic routes for blog posts:

- We want each post to have the path /posts/<id>
- getStaticPaths() returns an array of possible values for id

- [id].js this is a dynamic page
- you need to to tell nextjs what dynamic paths, page to create
- using getStatiPaths() you are telling nextjs that these are the defined paths to generate... else the amount of paths could be infinite...

- If a page has Dynamic Routes and uses getStaticProps, it needs to define a list of paths to be statically generated.
- When you export a function called getStaticPaths (Static Site Generation) from a page that uses dynamic routes, Next.js will statically pre-render all the paths specified by getStaticPaths.

- getStaticPaths must be used with getStaticProps. You cannot use getStaticPaths with getServerSideProps
- You can export getStaticPaths from a Dynamic Route that also uses getStaticProps
- in the example we are trying to find the 'id' for [id].js

-Important: The returned list is not just an array of strings — it must be an array of objects that look like the comment above. Each object must have the params key and contain an object with the id key (because we’re using [id] in the file name). Otherwise, getStaticPaths will fail.

## getStaticProps()

- fetches necessary data for the post with 'id'

- getStaticProps() will be called for each path object returned from getStaticPaths()

```js
// [id].js

export const getStaticPaths = async () => {
  const res = await fetch(
    'https://jsonplaceholder.typicode.com/users?_limit=3'
  );

  const data = await res.json(); //returns array of objects

  const paths = data.map((ninja) => {
    return {
      params: { id: ninja.id.toString() },
    };
  });

  return {
    paths, //each object in paths [] represents a route
    fallback: false, //shows 404 if no fallback
  };
};

export const getStaticProps = async (context) => {
  const id = context.params.id;

  const res = await fetch('https://jsonplaceholder.typicode.com/users/' + id);
  const data = await res.json();
  return { props: { ninja: data } };
};

export default function Page({ ninja }) {
  return (
    <div>
      <h1>{ninja.name}</h1>
      <h1>{ninja.email}</h1>
    </div>
  );
}
```

## test links:

```browser
http://localhost:3000/posts/ssg-ssr
http://localhost:3000/posts/pre-rendering
```

## Catch-all Routes

- https://nextjs.org/docs/pages/building-your-application/routing/dynamic-routes#catch-all-routes

- Dynamic routes can be extended to catch all paths by adding three dots (...) inside the brackets. For example:
- pages/posts/[...id].js matches /posts/a, but also /posts/a/b, /posts/a/b/c and so on.

- For example, pages/shop/[...slug].js will match /shop/clothes, but also /shop/clothes/tops, /shop/clothes/tops/t-shirts, and so on.

```
pages/shop/[...slug].js
eg url: /shop/a
params: { slug: ['a'] }


pages/shop/[...slug].js
eg url: /shop/a/b
params: { slug: ['a', 'b'] }


pages/shop/[...slug].js
eg url: /shop/a/b/c
params: { slug: ['a', 'b', 'c'] }
```

## Optional Catch-all Segments

Catch-all Segments can be made optional by including the parameter in double square brackets: [[...folderName]].

For example, pages/shop/[[...slug]].js will also match /shop, in addition to /shop/clothes, /shop/clothes/tops, /shop/clothes/tops/t-shirts.

The difference between catch-all and optional catch-all segments is that with optional, the route without the parameter is also matched (/shop in the example above).

## nextjs router

If you want to access the Next.js router, you can do so by importing the useRouter hook from next/router.

## custom 404

To create a custom 404 page, create pages/404.js. This file is statically generated at build time.

## Creating API Routes

- https://nextjs.org/docs/api-routes/introduction
  API Routes let you create an API endpoint inside a Next.js app. You can do so by creating a function inside the pages/api directory that has the following format:
- Do Not Fetch an API Route from getStaticProps or getStaticPaths
- A Good Use Case: Handling Form Input

### use cases

- https://nextjs.org/docs/pages/building-your-application/routing/api-routes#use-cases

```js
// req = HTTP incoming message, res = HTTP server response
export default function handler(req, res) {
  // ...
}
```

<!-- eg. put file in pages/api/hello.js -->

Try accessing it at http://localhost:3000/api/hello. You should see {"text":"Hello"}. Note that:

```js
// pages/api/hello.js
export default function handler(req, res) {
  res.status(200).json({ text: 'Hello' });
}
```
