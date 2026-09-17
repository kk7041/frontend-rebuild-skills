# Shell fit

Dashboard / product chrome that fights a reconstructed full-page UI.

## Padding island

`app/[locale]/dashboard/dashboard-layout-client.tsx` wraps pages in:

```
<main className="flex-1 p-0 sm:p-2 lg:p-4 ...">
```

A reconstructed page with `background: var(--bg-body)` (`#f8f9fa` plus a top glow) then sits inset on `bg-background`. Bleed with negative margin on the graft wrapper, not by editing the copied CSS module:

```
sm:-m-2 lg:-m-4
min-h-[calc(100svh-<header>)]
```

Header is `h-14 sm:h-16`. Promo banner `pt-20` on the outer shell may still leave a strip — measure if the user says it is not full.

## 100vh inside a framed app

Reconstructed `.main { height: 100vh }` is for a standalone preview. Inside the dashboard it overflows the app header. Scoped override:

```
.jimengFill {
  height: auto !important;
  min-height: 100%;
}
```

Do not change the copied module. Hashed CSS module class names cannot be targeted as `.main`.

## Tokens must be scoped

Preview `globals.css` / `:root` tokens will collide with the dashboard. Put tokens on `.jimeng-scope` (or equivalent), not `:root`.

## Composer dropdown under banners

Reconstructed `.composer { backdrop-filter }` creates a stacking context, so `.menu { z-index: 12 }` cannot paint over later siblings (What's new banners). Lift the wrap in the scoped sheet (`.jimengComposerLift { position: relative; z-index: 4 }`), do not edit the copied module.

## File input ref

Old hidden page and new composer cannot share the same `ref` on two `<input type="file">`. Keep one live input, usually next to the reconstructed upload tile.

## Generate-page hashed layout

`/ai-tool/generate` hashed CSS uses `height: 100vh` + `overflow: hidden` on `.entry-aq_KoZ` / `.feed-stage-zWr498` / `.content-wrapper-rXyMzu`, and `min-width` on `.main-container-_bf0Jm`. Dashboard layout adds `pt-20` for the promo banner and `SidebarProvider`/`SidebarInset` are `min-h-svh`, so a graft of `h-[100svh]` is taller than the remaining inset and the **window** scrolls — that is why the generate composer follows the page. Scene 2 wrapper must be `position: fixed` to the remaining inset (`top: var(--dashboard-banner-offset)`, `left: var(--sidebar-width)`), with only `.jm-record-feed` scrolling. Sticky/absolute inside a growing document still rides the window. Override `.entry-aq_KoZ { height: 100% !important }` so `100vh` does not overflow the app header. Do not edit the hashed sheet. Portal menus (`createPortal` to `body`) must also carry `.jimeng-scope` so scoped `:root .lv-btn` replacements still match. Neutralize `.jimeng-scope.jm-popup { width/min-height/background }` or Agent / Auto / Skill sit as a full-bleed sheet instead of a button-anchored menu.

If they name Jimeng left nav ("侧边栏删掉"), drop that node only and set `--side-menu-width: 0px`; keep dashboard chrome.

Home vs generate are two scenes. Default `/dashboard/text-to-image` to `JimengMain` ("你好，今天想要创作什么"). Switch to `JimengGenerate` only after a conversation starts. Do not restore the last conversation on load. Agent / Auto / Skill / @ live in `JimengComposerToolbar` and must be the same component on both scenes; do not keep a simplified home menu. Home menus open down (`placement="down"`); generate menus open up. Home `.main` must not keep `100vh` overflow; leave the dashboard/window scrollbar only. API Key sits next to 使用技能 in `JimengComposerToolbar`.

## Generate-page menus need the full preview CSS module + light tokens

Agent / 生成偏好 / 技能 / @ 弹层皮肤一半在 hashed CSS（`lv-select-popup`、`skill-select-dropdown`），一半在首页 CSS module（`.menu` / `.prefMenu` / `.slashMenu`）。只拷生成页 hashed 表、不全量覆盖 `JimengHome.module.css`，portal 里会变成无底无阴影的白条。`--shadow-dropdown-menu`、`--create-mode-motion-*`、`--lvv-color-*` 也要写进 scoped token 表，不要只抄 `globals.css` 那一小段。

## Generate-page 生成偏好 is not a native `<select>`

Preview `JimengGenerate` reconstructed Agent 偏好 with `<select>` as a shortcut. Live DOM from `content-generator-feature-loader-3` + chunk `5929` is `fields-F7kPxC` / `field-LiX7ZX` / `title-mDDLIr` + `lv-switch-small` + `trigger-button-I6Hqgr` (`model-select-cA_3Vx` / `resolution-select-jrUous`) opening `lv-select-popup`. Those field/title/option classes live in `static/css/async/5929.7c70b534eb.css`, which was not in `public/jimeng-generate.css`. Append that extract onto the scoped hashed sheet; do not restyle `.select` in the CSS module to fake it.
