---
name: public-frontend-rebuild
description: Rebuild a readable local frontend from a live site's public production bundle (HTML/CSS/JS/assets), not a private repo. Use when the user pastes a URL and wants the source, a local clone, "怎么做到的", "扒下来", "还原前端", "本地跑这个站", or a clickable preview on a given port.
---

Rebuild a **readable local preview** from what a browser already downloads. This is not cloning a private repo.

Default stance: local look / learn the technique. Do not treat the result as licensed product source.

## Hard gates

Stop and say so if any of these is true:

- The user wants backend, auth, checkout, private APIs, exploits, or bypassing paywalls.
- There is no public HTML/JS/CSS to read (empty shell that only talks to private APIs, or the interesting UI never ships to the client).

If they want the reconstructed look **in their own product**, finish this skill first, then invoke `graft-rebuilt-ui`. Do not graft during rebuild.

Otherwise continue. If GitHub / source maps later turn up, still treat them as a bonus — the public bundle is the primary source.

## Output contract

Unless the user names another path/port:

- Write the preview to `~/` + a short slug from the host (`ascend-studio`, `hf-quiz`).
- Run `next dev` on the first free port in `3000–3010` (or the port they named).
- Give them one local URL. State clearly: reconstructed preview, not original source.

## Steps

Do these in order. Each step has a checkable done-state.

### 1. Classify the page

Fetch the URL (bypass `http_proxy` for localhost later; for the live site, a proxy is fine).

Record:

- Title, framework signal (`/_next/`, `tanstack/assets`, `webpackChunk`, Vite hashed assets)
- Whether first HTML already contains the UI, or only a shell + route chunks
- Auth / API / checkout hints (Clerk, Stripe, `__TSR`, signed-out cookies)

Done when you can say **landing** (self-contained page) or **app-slice** (one route inside a product).

If **app-slice**, tell the user immediately: preview will cover the client-visible flow, not login/billing/save.

### 2. Hunt original source, then ignore the miss

In parallel:

- `gh search repos` / GitHub code search for distinctive copy
- Request `<js>.map` and look for `sourceMappingURL`

Done when you know: public repo, source maps, or neither. Missing both is normal — continue.

### 3. Pull the public bundle

Save HTML, CSS, JS, i18n JSON, and asset URLs under `/tmp/<slug>/`.

For SPAs, follow the **route graph**: HTML preload/module tags → router manifest → lazy `import('./chunk.js')` that actually mounts the page (look for `export { … as Quiz }` / `createFileRoute` / page component).

Done when the file that contains the page's JSX/strings is on disk, not just the 400-byte re-export stub.

Framework-specific extraction: [references/bundle-map.md](references/bundle-map.md).

### 4. Extract the interaction, not the whole app

From that file, recover only what paints the requested URL:

- Copy (string tables, i18n ids)
- Steps / routes / buttons
- CSS module hash map (`hero: "ascend_hero__Y9fQd"`)
- Public media URLs (probe CDN with `curl -I`)

Leave out: auth, analytics, checkout, private `fetch`, the rest of the product.

Done when you have a list of screens + options + media that a user can click through **offline of their API**.

If the UI is empty without a private JSON endpoint, stop — do not invent a fake product, and do not attack the API.

### 5. Rebuild a small runnable app

Default stack: Next.js app router + CSS module, matching whatever they already have locally if they asked to merge.

- Restore hashed CSS class names to locals
- Rename minified vars by role (`N` scroll progress, `droneX`, …)
- Point images/video at the **live public CDN** unless files are tiny enough to vendor
- README must say reconstructed-from-public-bundle

Done when `next build` or `next dev` serves the first screen with the original copy.

Port / proxy pitfalls: [references/local-preview.md](references/local-preview.md).

### 6. Hand the local link, with the caveat

Reply with:

1. The local URL
2. What was recovered vs what was not (repo, maps, APIs)
3. This is a reconstruction for looking / learning — original brand/copy/assets stay theirs

If they then say 接到我们的页面 / 移植 UI / 先复现整个 UI, invoke `graft-rebuilt-ui` with this preview path. That skill hides our old page, mounts this main surface, and wires existing APIs. Do not start grafting inside this skill.

## After a run

If you learned a new framework signal or CDN pattern, append one line to [references/bundle-map.md](references/bundle-map.md). Do not grow this SKILL.md.
