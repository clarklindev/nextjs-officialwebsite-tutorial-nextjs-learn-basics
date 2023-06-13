# Nextjs

- https://nextjs.org/learn/foundations/about-nextjs?utm_source=next-site&utm_medium=homepage-cta&utm_campaign=home

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

2. server side rendering (generates html at each request)
