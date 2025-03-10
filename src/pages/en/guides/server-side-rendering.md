---
layout: ~/layouts/MainLayout.astro
title: Server-side Rendering
i18nReady: true
---

**Server-side Rendering**, aka SSR, can be enabled in Astro. When you enable SSR you can:

- Implement sessions for login state in your app.
- Render data from an API called dynamically with `fetch`.
- Deploy your site to a host using an *adapter*.

## Enabling SSR in Your Project

To enable SSR you need to use an adapter. This is because SSR requires a server _runtime_: the environment that runs your server-side code. This runtime provides an API that your server-side code can use.

Installing an adapter gives Astro access to the corresponding API, and allows Astro to output a script that runs your project on that kind of server.

The following adapters are available today with more to come in the future:

- [Cloudflare](/en/guides/integrations-guide/cloudflare/)
- [Deno](/en/guides/integrations-guide/deno/)
- [Netlify](/en/guides/integrations-guide/netlify/)
- [Node.js](/en/guides/integrations-guide/node/)
- [Vercel](/en/guides/integrations-guide/vercel/)


You can add any of the official adapters with the following `astro add` command. This will install the adapter and make the appropriate changes to your `astro.config.mjs` file in one step. For example, to install the Netlify adapter, run:

```bash
npx astro add netlify
```

You can also add an adapter manually by installing the package and updating `astro.config.mjs` yourself. (See the links above for adapter-specific instructions to complete the following two steps to enable SSR.) Using `my-adapter` as an example placeholder, the instructions will look something like:

1. Install the adapter to your project dependencies using your preferred package manager. If you’re using npm or aren’t sure, run this in the terminal:

    ```bash
    npm install @astrojs/my-adapter
    ```
1. [Add the adapter](/en/reference/configuration-reference/) to your `astro.config.mjs` file's import and default export

    ```js ins={3,6-7}
    // astro.config.mjs
    import { defineConfig } from 'astro/config';
    import myAdapter from '@astrojs/my-adapter';

    export default defineConfig({
      output: 'server',
      adapter: myAdapter(),
    });
    ```

## Features

Astro will remain a static-site generator by default. But once you enable a server-side rendering adapter, **every route in your pages directory becomes a server-rendered route** and a few new features become available to you.

### `Astro.request.headers`

The headers for the request are available on `Astro.request.headers`. It is a [Headers](https://developer.mozilla.org/en-US/docs/Web/API/Headers) object, a Map-like object where you can retrieve headers such as the cookie.

```astro title="src/pages/index.astro" {2}
---
const cookie = Astro.request.headers.get('cookie');
// ...
---
<html>
  <!-- Page here... -->
</html>
```

:::caution
The features below are only available at the page level. (You can't use them inside of components, including layout components.)

This is because these features modify the [Response headers](https://developer.mozilla.org/en-US/docs/Glossary/Response_header), which can't be modified after they're sent to the browser. In SSR mode, Astro uses HTML streaming to send each component to the browser as it renders them. This makes sure the user sees your HTML as fast as possible, but it also means that by the time Astro runs your component code, it has already sent the Response headers.
:::

### `Astro.redirect`

On the `Astro` global, this method allows you to redirect to another page. You might do this after checking if the user is logged in by getting their session from a cookie.

```astro title="src/pages/account.astro" {8}
---
import { isLoggedIn } from '../utils';

const cookie = Astro.request.headers.get('cookie');

// If the user is not logged in, redirect them to the login page
if (!isLoggedIn(cookie)) {
  return Astro.redirect('/login');
}
---
<html>
  <!-- Page here... -->
</html>
```

### `Response`

You can also return a [Response](https://developer.mozilla.org/en-US/docs/Web/API/Response) from any page. You might do this to return a 404 on a dynamic page after looking up an id in the database.

```astro title="src/pages/[id].astro" {8-11}
---
import { getProduct } from '../api';

const product = await getProduct(Astro.params.id);

// No product found
if (!product) {
  return new Response(null, {
    status: 404,
    statusText: 'Not found'
  });
}
---
<html>
  <!-- Page here... -->
</html>
```

## API Routes

An [API route](https://medium.com/@rajat_m/what-are-restful-routes-and-how-to-use-them-929129ae7bf6) is a `.js` or `.ts` file within the `src/pages` folder that takes a [Request](https://developer.mozilla.org/en-US/docs/Web/API/Request) and returns a [Response](https://developer.mozilla.org/en-US/docs/Web/API/Response). A powerful feature of SSR, API routes are able to securely execute code on the server side.

### SSR and Routes

In Astro, these routes turn into server-rendered routes, allowing you to use features that were previously unavailable on the client side, or required explicit calls to a backend server and extra client code to render the results. 

In the example below, an API route is used to retrieve a product from a database, without having to generate a page for each of the options. 

```js title="src/pages/[id].js"
import { getProduct } from '../db';

export async function get({ params }) {
  const { id } = params;
  const product = await getProduct(id);

  if (!product) {
    return new Response(null, {
      status: 404,
      statusText: 'Not found'
    });
  }

  return new Response(JSON.stringify(product), {
    status: 200
  });
}
```

In this example, a valid HTML code can be returned to render the whole page or some of its content.


In addition to content fetching and server-side rendering, API routes can be used as REST API endpoints to run functions such as authentications, database access, and verifications without exposing sensitive data to the client.

In the example below, an API route is used to verify Google reCaptcha v3 without exposing the site-secret to the clients.


```astro title="src/pages/index.astro"
<html>
  <head>
    <script src="https://www.google.com/recaptcha/api.js"></script>
  </head>

  <body>
    <button class="g-recaptcha" 
      data-sitekey="PUBLIC_SITE_KEY" 
      data-callback="onSubmit" 
      data-action="submit"> Click me to verify the captcha challenge! </button>

    <script is:inline>
      function onSubmit(token) {
        fetch("/recaptcha", {
          method: "POST",
          body: JSON.stringify({ recaptcha: token })
        })
        .then((response) => response.json())
        .then((gResponse) => {
          if (gResponse.success) {
            // Captcha verification was a success
          } else {
            // Captcha verification failed
          }
        })
      }
    </script>
  </body>
</html>
```

In the API route you can safely define secret values, or read your secret environment variables.

```js title="src/pages/recaptcha.js"
import fetch from 'node-fetch';

export async function post({ request }) {
  const data = await request.json();

  const recaptchaURL = 'https://www.google.com/recaptcha/api/siteverify';
  const requestBody = {
    secret: "YOUR_SITE_SECRET_KEY",   // This can be an environment variable
    response: data.recaptcha          // The token passed in from the client
  };

  const response = await fetch(recaptchaURL, {
    method: "POST",
    body: JSON.stringify(requestBody)
  });

  const responseData = await response.json();

  return new Response(JSON.stringify(responseData), { status: 200 });
}
```

### Redirects

Since `Astro.redirect` is not available in API Routes you can use [`Response.redirect`](https://developer.mozilla.org/en-US/docs/Web/API/Response/redirect).

```js title="src/pages/links/[id].js" {14}
import { getLinkUrl } from '../db';

export async function get({ params }) {
  const { id } = params;
  const link = await getLinkUrl(id);

  if (!link) {
    return new Response(null, {
      status: 404,
      statusText: 'Not found'
    });
  }

  return Response.redirect(link, 307);
}
```

`Response.redirect` requires that you pass a full URL. For local redirects, you can use `request.url` as the base with [the `URL` constructor](https://developer.mozilla.org/en-US/docs/Web/API/URL/URL) to build an absolute URL:

```js title="src/pages/redirect.js"
export async function get({ request }) {
  const url = new URL('/home', request.url);
  return Response.redirect(url, 307);
}