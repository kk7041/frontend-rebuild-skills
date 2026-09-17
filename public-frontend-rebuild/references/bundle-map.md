# Bundle map

How to find the page component inside a production frontend. Read this in step 3.

## Next.js App Router

Signals: `/_next/static/chunks/`, `self.__next_f`, `next-server`.

1. HTML lists CSS + `app/page-….js` (often a tiny stub).
2. The stub names sibling chunks (`825`, `153`, …). Download those.
3. Look for `e.exports = { hero: "ascend_hero__Y9fQd", … }` — that is the CSS Modules map.
4. The same chunk usually has `useState` / `useRef` / `useEffect` and `jsx(` / `jsxs(` for the page.
5. Pretty-print CSS, replace hashed names via the map, restore `@keyframes` names the same way.

Source maps: `/_next/static/chunks/<file>.js.map`. 404 is typical on Firebase/Vercel prod.

Alibaba Marmot / Tbox portal (`cdn.marmot-cloud.com/page/<app>/_next/`, `x-powered-by: Next.js`, `x-server-id: tboxrouter-portal-*`): same App Router hunt. `app/models/page-*.js` already contains the page + CSS module maps (`ModelList_container__VuMeR`). Sibling `76306-*.js` is `ModelItem`. Public catalog JSON is already in the browser: `/api/frontend/model/listByFilter`, `/api/frontend/model/modelAuthors`, `/api/frontend/provider/list`. Snapshot those instead of inventing rows. Icons live on `cdn.marmot-cloud.com/storage/zenmux/…`.

## TanStack Start / Vite SPA

Signals: `x-hf-worker: tanstack`, `$_TSR.router`, `assets.example.com/tanstack/assets/<buildId>-<hash>.js`.

1. HTML `$_TSR.router.manifest.routes` names the route (`/_private/quiz`) and its `preloads`.
2. The first preload is often a route wrapper. Search it for `import('./….js').then(e => e.Quiz)` (or the page export).
3. The 400-byte file that `export { t as Quiz }` is a barrel — download the file it imports (the real 100k+ component).
4. Copy often lives in an i18n JSON (`/_i18n/en.<hash>.json`) keyed by short ids (`id:\`GqV9EU\``). Resolve those ids; do not ship hashed keys in the preview.
5. Media helpers look like `j('quiz-v2/updated/q1-upd', 'mp4')`. Probe `https://static.<host>/` + path + ext with `curl -I` until you get `200` + the right `content-type`.

No `sourceMappingURL` is normal. Do not download the entire 10MB global CSS unless tokens are missing — reconstruct layout CSS from className strings (`rounded-xl`, `bg-surface-secondary`) plus a few measured colors from the live page.

## ByteDance Modern.js / EdenX SSR (Jimeng / Dreamina)

Signals: `window._SSR_DATA`, `window._ROUTER_DATA`, `id="ssr-root"`, `modern-js-R…` hydration ids, `/static/js/async/ai-tool/home/page.<hash>.js`, `edenx-sidecode`.

1. First HTML already contains the painted UI (`ssr-root`) plus `window._ROUTER_DATA.loaderData['ai-tool/home/page']` (banners, feed, categories).
2. Dark tokens live in an inline `<style>` as `html [lv-theme=dark],html body[lv-theme-version="2.0"]{--bg-body:#121212;…}` — copy that block, do not hunt hashed CSS for `--text-primary`.
3. Layout CSS is split: shell in another inline `<style>` (`sidebar-byjpfT`), page in `/static/css/async/4700.<hash>.css` and `Home.<hash>.css`. Source maps 404.
4. Rebuild the visible home slice (sidebar / composer / whats-new / feed) from SSR + loader JSON. Leave login, generate APIs, canvas, billing out.
5. Cover thumbs in SSR are 200px; `commonAttr.coverUrlMap['1080']` is the usable still.
6. **Do not stop at named page chunks.** Parse `runtime.*.js` `l.u` (JS) and `miniCssF` (CSS) — that is the full async graph (Jimeng: 293 JS + 90 CSS). HTML only lists what the current route preloads. Numbered ids are either filenames (`6470.…js`) or **module ids inside a file** (`765424(e,r,n){…}` in `6470.…js`). Search downloaded JS for the id before assuming a missing file. User-bubble (Jimeng): module `765424`, file `static/js/async/6470.*.js` + `static/css/async/6470.*.css`, classes `user-message-yM70Oj` / `bubble-JU_QFn` / `user-message-text-lgHpnm`.
7. Source maps 404. Incomplete local preview is usually "we only grabbed named chunks + rewrote JSX", not "the site hid the bundle".

## Generic webpack / Vite

- Webpack: `self.webpackChunk…push([[id], { … })` — find the module that contains unique page copy.
- Vite: hashed filenames in `import.meta.url` + `__vite__mapDeps`. Follow the dep that is unique to the route, not the shared vendor file.

## What "the right file" looks like

Keep going until the file contains **page copy or JSX**, not:

- Sentry debug id stubs
- `export { t as Page }`
- A 10k+ vendor (`phosphor-icons`, `framer-motion` runtime) with no product strings

## CDN probing

Try in this order, HEAD only:

1. Origin in the HTML (`https://static.example.com/…`)
2. Path as written in JS (`quiz-v2/updated/q1-upd.mp4`)
3. Common prefixes: `/public/`, `/assets/`, CloudFront hosts already in the HTML

Stop at the first `200` with `image/*`, `video/*`, or `font/*`. Vendor into `public/` only if the file is small (< ~200KB) or the CDN is likely to hotlink-block localhost.

## Cloudflare Dashboard (dash.cloudflare.com, Vite + kumo)

Signals: `/assets/app.<hash>.js`, `/assets/app.<hash>.css`, `aside.group/sidebar`, `--color-kumo-*`, `/assets/combined-icons.<hash>.svg`.

1. Anonymous curl hits a managed challenge (403 HTML `Just a moment...`). Use an already-logged-in Chrome tab and `fetch()` same-origin assets.
2. Sidebar DOM is client-rendered after auth. Extract `aside` + `data-testid=sidebar-nav-*` + sprite `<use href="#icon-...">`.
3. Tokens live in `app.<hash>.css` (`--color-kumo-canvas`, `--color-kumo-tint`, `--color-kumo-line`). Font: `/fonts/inter-variable.woff2`.
4. Rebuild only the chrome that is visible without private APIs. Leave login / billing / GraphQL out.
