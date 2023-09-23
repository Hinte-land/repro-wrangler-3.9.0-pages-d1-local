# Minimal repro

...of an issue with Wrangler 3.9.0 when using Pages with local D1 binding.

## Step by step

- `gh repo create Hinte-land/repro-wrangler-3.9.0-pages-d1-local --public --clone`
- `cd repro-wrangler-3.9.0-pages-d1-local`
- `echo ".DS_Store\n.wrangler\nnode_modules" > .gitignore`
- `pnpm init`
- `pnpm i -D wrangler` // wrangler 3.9.0
- `mkdir public && echo "hello world" > public/index.html`
- `pnpm wrangler pages dev ./public` // check http://127.0.0.1:8788; kill the process
- `pnpm wrangler d1 create repro` // paste output to `wrangler.toml`
- `git add -A; git commit -m "Initial commit"; git push -u origin main`
- Create application from the Workers & Pages Overview in the Cloudflare Dashboard, using `Hinte-land/repro-wrangler-3.9.0-pages-d1-local`, no build, with output in `public` // deployed to <https://repro-wrangler-3-9-0-pages-d1-local.pages.dev>
- As per <https://developers.cloudflare.com/pages/platform/functions/bindings/#d1-databases>, create D1 bindings (`DB = repro`) for both production and preview environments
- `mkdir functions`
- `echo "export function onRequestGet(context) { return new Response(Date.now()) }" > functions/index.js`
- `pnpm wrangler pages dev ./public` // check http://127.0.0.1:8788; should show timestamp
- `echo "export function onRequestGet(context) { return new Response('Hello') }" > functions/index.js` // a simple change to the worker while `pnpm wrangler pages dev ./public` is running

The final step causes Wrangler to crash with the following output:

```
/Users/andy/§/github.com/hinte-land/repro-wrangler-3.9.0-pages-d1-local/node_modules/.pnpm/wrangler@3.9.0/node_modules/wrangler/wrangler-dist/cli.js:30947
            throw a;
            ^

Error [ERR_IPC_CHANNEL_CLOSED]: Channel closed
    at new NodeError (node:internal/errors:405:5)
    at target.send (node:internal/child_process:754:16)
    at Object.announceAndOnReady [as onReady] (/Users/andy/§/github.com/hinte-land/repro-wrangler-3.9.0-pages-d1-local/node_modules/.pnpm/wrangler@3.9.0/node_modules/wrangler/wrangler-dist/cli.js:150032:15)
    at MiniflareServer.<anonymous> (/Users/andy/§/github.com/hinte-land/repro-wrangler-3.9.0-pages-d1-local/node_modules/.pnpm/wrangler@3.9.0/node_modules/wrangler/wrangler-dist/cli.js:128172:24)
    at process.processTicksAndRejections (node:internal/process/task_queues:95:5)
Emitted 'error' event on process instance at:
    at process.processEmit [as emit] (/Users/andy/§/github.com/hinte-land/repro-wrangler-3.9.0-pages-d1-local/node_modules/.pnpm/wrangler@3.9.0/node_modules/wrangler/wrangler-dist/cli.js:25324:38)
    at node:internal/child_process:758:35
    at process.processTicksAndRejections (node:internal/process/task_queues:77:11) {
  code: 'ERR_IPC_CHANNEL_CLOSED'
}

Node.js v20.7.0
```
