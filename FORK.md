# This is a public mirror of `Soul-Brews-Studio/maw-ui`

Same UI, same features, same 143 commits of history — deployed separately so it
can be pointed at a maw node of your choosing.

- **upstream**: https://github.com/Soul-Brews-Studio/maw-ui (already public)
- **original deployment**: https://god.buildwithoracle.com
- **this deployment**: https://arra-office.laris.workers.dev

Upstream is treated as read-only. Changes belong there, not here; this repo
exists to be deployed, not to diverge.

## The thing everyone gets backwards: there is no backend at the URL

`god.buildwithoracle.com` serves **static assets and nothing else**. Every
`/api/*` path there returns **404** — measured, all of them:

```
/api/identity  /api/sessions  /api/teams  /api/feed  /api/config
/api/costs     /api/plugins   /api/worktrees  /api/digest  /api/federation/status
                          → 404, every one
```

`wrangler.god.json` has no `main`, so the Worker has zero compute. The same is
true of this deployment by design.

**The backend is your own maw node.** The browser talks directly to
`maw-rs` / `maw-js` on `:3456`, and the CDN never sees a request or a token.
Verified against m5's daemon, which answers the whole surface:

```
:3456/api/identity 200 · /api/sessions 200 · /api/teams 200
:3456/api/feed 200 · /api/federation/status 200
```

So "use the hosted backend" is not a thing that exists. Open the page, point it
at a node you run, and the page is just the renderer.

## Choosing the node

In the UI — it is canonicalized and stored per browser.

**Do not reintroduce `?host=`.** It was removed deliberately: an import-time
`?host=` consumer let an attacker choose the origin the page sends credentials
to. `src/lib/apiMigration.test.ts` has a regression test whose entire job is to
fail if anyone adds one back. Respect it.

## What this deployment includes

13 pages, all verified serving 200:

`/` · `/office` · `/federation` · `/federation_2d` · `/fleet` · `/mission`
`/terminal` · `/chat` · `/config` · `/inbox` · `/overview` · `/workspace` · `/dashboard`

Cloudflare drops the `.html` extension, so `/office.html` 307s to `/office`.

`arena.html`, `shrine.html`, `talk.html` and `timemachine.html` exist in the
source but are **not in vite's `rollupOptions.input`**, so upstream does not
build or deploy them either. Left as-is rather than silently adding pages the
original does not have.

## Deploy

```sh
bun install && bun run build
npx wrangler deploy --config wrangler.jsonc
```

## A security observation, unmodified from upstream

The maw daemon reflects **any** `Origin` back in `access-control-allow-origin`
and sets `access-control-allow-private-network: true`, and `/api/identity`
answers **200 with no credential** — it returns the full agent roster. That means
any web page in a browser that can route to the daemon can enumerate the fleet.

This is upstream behaviour and is not changed here; it is written down because a
second public deployment widens who might notice it. The fix belongs in `maw-rs`,
not in this renderer.
