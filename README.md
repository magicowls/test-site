# feedbakkr-test-site

A tiny static site for manually smoke-testing the
[`@magicowls/feedbakkr-js`](https://www.npmjs.com/package/@magicowls/feedbakkr-js)
embed on a real hosted origin.

## Why it exists

Testing a reporter-side embed from `file://` or a local `python -m http.server`
hits a wall of origin / `null`-origin / CORS restrictions. A real hosted
origin on Netlify (or anywhere else) side-steps all of that and lets the
admin allowed_origins list behave normally.

## Layout

- `index.html` — a handful of elements in different shapes (headings, lists,
  form fields, an image, deeply-nested text) so we can verify the picker +
  anchoring land on sensible selectors
- `feedbakkr.iife.js` — vendored build of the vanilla SDK. Until the SDK is
  published on npm, this is how we keep the test page self-contained. Rebuild
  from the feedbakkr monorepo with `pnpm --filter @magicowls/feedbakkr-js
  build` and `cp packages/sdk-vanilla/dist/feedbakkr.iife.js` here.

## Deploy (Netlify)

1. Push this repo to GitHub under the `magicowls` org:
   `git remote add origin git@github.com:magicowls/feedbakkr-test-site.git`
   `git push -u origin main`
2. In Netlify: **Add new site → Import from Git**, pick the repo, no build
   command, publish directory `.`
3. Note the Netlify URL (e.g. `magicowls-test.netlify.app`).

## Provision a feedbakkr site

1. Sign in to [admin.feedbakkr.io](https://admin.feedbakkr.io).
2. Create a new site (e.g. "vanilla SDK test"). Set **Allowed origins** to
   `https://magicowls-test.netlify.app` (whatever Netlify gave you).
3. Copy the public key (`fbk_pk_…`).
4. In `index.html` replace `REPLACE_WITH_YOUR_SITE_KEY` with the public key.
   Commit, push, Netlify redeploys automatically.

## Regenerate the embed

When the vanilla SDK changes in the monorepo:

```bash
cd ~/Projects/feedbakkr/packages/sdk-vanilla
pnpm build
cp dist/feedbakkr.iife.js ~/Projects/test-site/feedbakkr.iife.js
cd ~/Projects/test-site && git add feedbakkr.iife.js && git commit -m "bump iife"
git push
```

Once the SDK is on npm we can switch `index.html` to load from unpkg:

```html
<script src="https://unpkg.com/@magicowls/feedbakkr-js@latest/dist/feedbakkr.iife.js"
        data-site-id="fbk_pk_..."
        data-api-base="https://api.feedbakkr.io" defer></script>
```
