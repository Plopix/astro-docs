---
title: Deploy your Astro Site to Cloudflare Pages
description: How to deploy your Astro site to the web using Cloudflare Pages.
layout: ~/layouts/DeployGuideLayout.astro
i18nReady: true
---

You can deploy your Astro project on [Cloudflare Pages](https://pages.cloudflare.com/), a platform for frontend developers to collaborate and deploy static (JAMstack) and SSR websites.

This guide includes instructions for deploying to Cloudflare Pages through the Pages Dashboard or using Wrangler the Cloudflare CLI.

## Prerequisites

To get started, you will need:

- A Cloudflare account. If you don’t already have one, you can create a free Cloudflare account during the process.
- Your app code pushed to a [GitHub](https://github.com/) or a [GitLab](https://about.gitlab.com/) repository.

## How to deploy a site with Git

1. Set up a new project on Cloudflare Pages.
2. Push your code to your git repository (GitHub, GitLab).
3. Log in to the Cloudflare dashboard and select your account in **Account Home** > **Pages**.
4. Select **Create a new Project** and the **Connect Git** option.
5. Select the git project you want to deploy and click **Begin setup**
6. Use the following build settings:

    - **Framework preset**: `Astro`
    - **Build command:** `npm run build`
    - **Build output directory:** `dist`
    - **Environment variables (advanced)**: By default, Cloudflare Pages uses Node.js 12.18.0, but Astro [requires a higher version](/en/install/auto/#prerequisites). Add an environment variable with a **Variable name** of `NODE_VERSION` and a **Value** of `v16.13.0` or higher to tell Cloudflare to use a compatible Node version. Alternatively, add a `.nvmrc` file to your project to specify a Node version.

7. Click the **Save and Deploy** button.

## How to deploy a site using Wrangler

1. Install [Wrangler CLI](https://developers.cloudflare.com/workers/wrangler/get-started/).
2. Authenticate Wrangler with your Cloudflare account using `wrangler login`.
3. Run your build command.
4. Deploy using `npx wrangler pages publish dist`.

```bash
# Install Wrangler CLI
npm install -g wrangler
# Login to Cloudflare account from CLI
wrangler login
# Run your build command
npm run build
# Create new deployment
npx wrangler pages publish dist
```

After your assets are uploaded, Wrangler will give you a preview URL to inspect your site. When you log into the Cloudflare Pages dashboard, you will see your new project.

### Enabling Preview locally with Wrangler

For the preview to work, you must install `wrangler`

```bash
pnpm install wrangler --save-dev
```

It's then possible to update the preview script in your `package.json` to `"preview": "wrangler pages dev ./dist"`

## How to deploy an SSR site

You can also deploy an Astro SSR site to Cloudflare Pages using the [`@astrojs/cloudflare` adapter](/en/guides/integrations-guide/cloudflare/).

Add the Cloudflare adapter to enable SSR in your Astro project with the following `astro add` command. This will install the adapter and make the appropriate changes to your `astro.config.mjs` file in one step.

```bash
npx astro add cloudflare
```

If you prefer to install the adapter manually instead, complete the following two steps:

1. Add the `@astrojs/cloudflare` adapter to your project's dependencies using your preferred package manager. If you’re using npm or aren’t sure, run this in the terminal:

```bash
npm install @astrojs/cloudflare
```

2. Add the following to your `astro.config.mjs` file:

```js title="astro.config.mjs" ins={2, 5-6}
import { defineConfig } from 'astro/config';
import cloudflare from '@astrojs/cloudflare';

export default defineConfig({
  output: 'server',
  adapter: cloudflare()
});
```

### Modes

There are currently two modes supported when using Pages Functions with the [`@astrojs/cloudflare`](https://github.com/withastro/astro/tree/main/packages/integrations/cloudflare#readme) adapter. 

1. **Advanced** mode: This mode is used when you want to run your function in `advanced` mode which picks up the `_worker.js` in `dist`, or a directory mode where pages will compile the worker out of a functions folder in the project root.  

> If no mode is set, the default  is `"advanced"`

2. **directory** mode: This mode is used when you want to run your function in `directory` mode, which means the adapter will compile the client side part of you app the same way, but it will move the worker script into a `functions` folder in the project root. The adaptor will only ever place a `[[path]].js` in that folder, allowing you to add additional plugins and pages middleware which can be checked into version control.

```ts title="astro.config.mjs" "directory"
export default defineConfig({
  adapter: cloudflare({ mode: "directory" }),
});
```
### Using Pages Functions

[Pages Functions](https://developers.cloudflare.com/pages/platform/functions/) enable you to run server-side code to enable dynamic functionality without running a dedicated server.

To get started, create a `/functions` directory at the root of your project. Writing your Functions files in this directory automatically generates a Worker with custom functionality at the predesignated routes. To learn more about writing Functions, see the [Pages Functions documentation](https://developers.cloudflare.com/pages/platform/functions/).

📚 Read more about [SSR in Astro](/en/guides/server-side-rendering/).

## Troubleshooting

If you're encountering errors, double-check the version of `node` you're using locally (`node -v`) matches the version you're specifying in the environment variable.

Cloudflare requires [node `v16.13`](https://miniflare.dev/get-started/cli#installation), which is a more recent version than Astro’s out-of-the-box minmimum, so double check you’re using at least `v16.13`.
