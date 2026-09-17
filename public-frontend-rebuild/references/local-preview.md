# Local preview

Step 5–6 mechanics.

## Project shape

```
~/<slug>/
  app/page.tsx          # mounts the reconstructed component
  app/layout.tsx
  app/globals.css       # reset + tokens
  components/<Name>.tsx # the page
  components/<name>.module.css
  public/               # only small/local assets
  README.md             # reconstructed-from-public-bundle + original URL
```

Keep one client component unless the live page is clearly several routes. For a quiz, `quiz-data.ts` (steps/options/media) + `QuizApp.tsx` is enough.

## Port

```bash
# first free port, or the one the user named
lsof -nP -iTCP:3000 -sTCP:LISTEN
```

This machine often already has Next apps on 3000/3001. Do not kill them unless asked. Pick the next free port.

Start detached so the reply can include a live URL:

```bash
cd ~/<slug> && nohup npm run dev -- -p <port> >/tmp/<slug>-dev.log 2>&1 &
sleep 3
curl -x '' -o /dev/null -w "%{http_code}\n" http://127.0.0.1:<port>/
```

## Proxy

Shell may have `http_proxy=http://127.0.0.1:7890`. `curl http://localhost:<port>` then hits the proxy and returns **502**. Always:

```bash
curl -x '' http://127.0.0.1:<port>/
```

Browsers on this Mac usually bypass proxy for localhost; still give `http://localhost:<port>`, not the LAN IP, unless they ask.

## Next workspace root warning

A `package-lock.json` in `$HOME` makes Next 16 pick the wrong turbopack root. Set in `next.config.mjs`:

```js
turbopack: { root: process.cwd() }
```

## Completion check

- `lsof` shows LISTEN on the chosen port
- `curl -x ''` returns 200 and the original `<title>` or first heading
- First screen shows original copy; media either loads from public CDN or a local file
