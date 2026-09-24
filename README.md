# LessPass (static mirror)

A static, self-hosted mirror of the **LessPass web app**, deployed to GitHub Pages.
No backend, no account, no sync — password generation is 100% client-side
(PBKDF2 of site + login + master password), exactly like the official app.

- **Live:** https://coattails-droid.github.io/lesspass-live/
- **Upstream source:** https://github.com/lesspass/lesspass
- **Upstream commit built:** `375ea859c932550926dbd1af04f4d94c17b73dd1`
- **Build date:** 2026-09-24
- **Build commands** (run from the upstream checkout; the repo's CONTRIBUTING.md
  says `yarn install && yarn build`, but yarn's fetcher could not complete
  through this environment's egress proxy, so pnpm — which natively understands
  the repo's `workspace:*` protocol — was used for dependency installation):
  ```sh
  git clone https://github.com/lesspass/lesspass.git   # commit 375ea859c932550926dbd1af04f4d94c17b73dd1
  cd lesspass
  pnpm install   # installs the web-site + web-component (+ core/i18n/api) workspace closure
  # build the shared web-component library first, with the Pages subpath baked in:
  ./node_modules/.bin/vite build --base=/lesspass-live/   # in packages/lesspass-web-component
  # then the site itself (i18n step copies translation files into public/):
  cd packages/lesspass-web-site
  rm -rf ./public/locales && mkdir -p ./public/locales \
    && cp -r ../lesspass-i18n/src/locales/* ./public/locales/
  ./node_modules/.bin/vite build --base=/lesspass-live/
  ```
  `--base=/lesspass-live/` adapts all asset URLs to the GitHub Pages subpath.
- **Deploy contents:** the Vite `dist/` output of `packages/lesspass-web-site`,
  plus `404.html` (a copy of `index.html` so client-side routes work on Pages),
  `.nojekyll`, this README, and the upstream license.

## Build-time patches (vs upstream commit)

Two minimal source patches were applied so the app works when served from the
`/lesspass-live/` subpath instead of a domain root:

1. `packages/lesspass-web-component/src/LessPassWebComponent.tsx` —
   `<BrowserRouter>` → `<BrowserRouter basename={import.meta.env.BASE_URL}>`
   so React Router matches routes under `/lesspass-live/`.
2. `packages/lesspass-web-component/src/i18n.ts` — set the i18next-http-backend
   `loadPath` to `${import.meta.env.BASE_URL}locales/{{lng}}/{{ns}}.json`
   so translation files load from the subpath.

Both values are baked in at build time from the same `--base=/lesspass-live/`
flag. No behavior changes beyond subpath hosting.

## Notes

- Core password generation needs **no backend**: it runs entirely in your
  browser. The optional LessPass account features (sign in, synced profiles)
  point at the upstream API (`https://api.lesspass.com`) and are unused here.
- Upstream license: **GPL-3.0** (see `LICENSE`). Note that the upstream
  `packages/lesspass-web-component` package declares its own license as
  **AGPL-3.0-only**; both texts are referenced here for completeness.
- This is a static mirror for convenience; the canonical project lives at
  https://github.com/lesspass/lesspass.
