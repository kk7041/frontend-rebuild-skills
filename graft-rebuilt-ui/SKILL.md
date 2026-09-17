---
name: graft-rebuilt-ui
description: Graft a reconstructed public-frontend preview into tbug's own product page. Use when the user points at a rebuilt preview (Desktop/jimeng, ~/hf-quiz, a local clone) and says 接到我们的页面, 移植 UI, 搞到文生图, 先复现整个 UI, 能接的接口都接上; or after public-frontend-rebuild when they want the look in their repo.
---

Take a **local reconstructed preview** from `public-frontend-rebuild` and mount it on an existing product page. Preview first, product second. Do not rebuild from the live URL here — if there is no preview yet, invoke `public-frontend-rebuild` first.

This is the second hop. Rebuild = look / learn. Graft = put that look on our page and wire what we already have.

## Hard gates

Stop and say so if any of these is true:

- There is no reconstructed preview on disk, and the user did not give a live URL to rebuild first.
- They want the original site's private APIs, login, billing, or a bypass.
- The target page's existing generate / upload / history logic would be deleted. Hide it; do not delete it.

## Output contract

- Target page shows the reconstructed **main surface** (composer, banners, feed, tokens, brand assets they said to keep).
- Reconstructed CSS / markup stay byte-faithful. Do not restyle, rename classes, or "improve" spacing.
- Our old page stays in the tree, wrapped `hidden` (or equivalent). Logic (generate, poll, upload, history) stays callable.
- Only existing product APIs get wired. Missing backends keep the reconstructed control and a toast / notice.

## Steps

Do these in order.

### 1. Confirm both ends

Need:

- Preview root (e.g. `~/Desktop/jimeng`) with `components/`, `*.module.css`, `data/`, `public/`
- Target page (e.g. `app/[locale]/dashboard/text-to-image/page.tsx`)
- What to drop from the reconstructed chrome (default: **their** left nav / product sidebar)
- What to keep of ours (default: keep our controls **hidden** with the old page; only mount the reconstructed main)

If the preview is missing, invoke `public-frontend-rebuild`, then come back.

Done when both paths exist and you can name the chrome to drop in one sentence.

### 2. Vendor, do not rewrite

Copy into the product repo under a scoped folder (`components/<feature>/<slug>/`, `public/<feature>/<slug>/`):

- CSS module **as-is** (`cmp` against the preview file after copy)
- icons, data JSON, tokens
- small local assets (logo)
- a scoped token sheet (`jimeng.css`) so `--bg-body` etc. do not leak into the dashboard

Do **not** edit the copied CSS module to fit the dashboard. Layout friction lives in the scoped sheet or the page wrapper.

Done when `cmp` says the CSS module matches the preview, and the product can import it.

### 3. Hide ours, mount theirs

On the target page:

1. Keep the existing JSX tree. Wrap the old return in a `hidden` container. Do not strip handlers, dialogs, or state.
2. Render the reconstructed **main** next to it (composer + feed). Omit the reconstructed product sidebar unless the user asked to keep it.
3. Point reconstructed actions at existing handlers: generate, upload, drop files, prompt state.

If they later say "原来的操作栏也不要了", hide our controls too — still do not delete them.

Done when the visible page is the reconstructed main, and the old tree still compiles behind `hidden`.

### 4. Wire only what already exists

Map reconstructed controls → product APIs:

| Reconstructed control | Wire if we have | Otherwise |
| --- | --- | --- |
| Send / Enter | create-task + poll | disable + toast |
| Upload tile / drop | `/api/user/image-upload` (or equivalent) | notice, no fake upload |
| Same-style / 同款 | fill prompt from card title | notice |
| Agent / Auto / Skill / Canvas / extra tabs | leave UI | toast "后端还没接" |
| Feed category tabs beyond shipped JSON | leave UI | notice |

Do not invent endpoints. Do not restyle the reconstructed composer to look like our old input box.

Done when generate / upload / history-restore still hit the same URLs as before, and unwired menus only toast.

### 5. Bleed the reconstructed canvas

Dashboard shells often pad `main` (`p-2` / `lg:p-4`) and paint `bg-background`. The reconstructed page then sits on a white island, and `--bg-body` / the hero glow look "not full bleed".

Fix on the **wrapper**, not in their CSS module:

- Negative margin equal to the dashboard `main` padding (`sm:-m-2 lg:-m-4`)
- Wrapper `min-height` fills the inset under the dashboard header
- Override reconstructed `height: 100vh` on the inner main via a scoped class (`height: auto; min-height: 100%`) so it does not overflow the dashboard header

Done when the reconstructed background meets the dashboard header and the app sidebar, with no white gutter.

### 6. Cut chrome only when asked

Default graft keeps reconstructed generate/canvas tabs, toolbars, banners, feed.

If they name a piece ("生成和画布 tab 删了", "左侧栏不要"), remove **that node only**. Do not take it as license to restyle the rest.

Done when the named chrome is gone and the remaining reconstructed tree still matches the preview.

## After a run

If a new dashboard-shell gotcha shows up (padding, header height, overflow), append one line to [references/shell-fit.md](references/shell-fit.md). Do not grow this SKILL.md.
