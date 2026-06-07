# Setup & Integration

## Two Integration Methods

| | AI Binding | REST API |
|--|-----------|---------|
| Recommended | Yes | Only outside Cloudflare infra |
| Token needed | No | Yes (`CLOUDFLARE_AI_TOKEN`) |
| Account ID needed | No | Yes (`CLOUDFLARE_ACCOUNT_ID`) |
| Quota tracking | Accurate (matches dashboard) | Separate -- may show 0 in dashboard while still returning 429 |
| Latency | Lower (no extra HTTP hop) | Higher |
| Local dev | `wrangler dev --remote` | Any HTTP client |

---

## Method 1: AI Binding (Recommended)

### wrangler.jsonc

```jsonc
{
  "ai": { "binding": "AI" }
}
```

That is the only required change. No secrets, no account ID in code.

### In a plain Cloudflare Worker

```ts
export default {
  async fetch(request: Request, env: Env): Promise<Response> {
    const result = await env.AI.run("@cf/qwen/qwen3-30b-a3b-fp8", {
      messages: [
        { role: "system", content: "You are a helpful assistant." },
        { role: "user", content: "Hello" },
      ],
    })
    return Response.json(result)
  },
}
```

### In Next.js on Cloudflare Pages (via OpenNext)

Use `getCloudflareContext` to access the binding from API routes or Server Components:

```ts
import { getCloudflareContext } from "@opennextjs/cloudflare"

export async function callAI(system: string, user: string): Promise<string> {
  const { env } = await getCloudflareContext()

  const result = await env.AI.run("@cf/qwen/qwen3-30b-a3b-fp8", {
    messages: [
      { role: "system", content: system },
      { role: "user", content: user },
    ],
  }) as { response?: string; choices?: Array<{ message: { content: string } }> }

  const text = result.response ?? result.choices?.[0]?.message?.content ?? ""
  if (!text) throw Object.assign(new Error("Empty AI response"), { status: 500 })
  return text.trim()
}
```

### TypeScript types

```bash
npm install --save-dev @cloudflare/workers-types
```

In `tsconfig.json`:
```json
{
  "compilerOptions": {
    "types": ["@cloudflare/workers-types"]
  }
}
```

Or declare the binding inline if you prefer not to add the package:
```ts
declare const AI: {
  run(model: string, options: { messages: Array<{ role: string; content: string }> }): Promise<unknown>
}
```

---

## Method 2: REST API

Use only when running outside Cloudflare infrastructure (e.g., a local Node.js server, a different cloud provider).

### Secrets required

```bash
wrangler secret put CLOUDFLARE_ACCOUNT_ID
wrangler secret put CLOUDFLARE_AI_TOKEN  # must have "Workers AI: Run" permission
```

### Implementation

```ts
const MODEL = "@cf/qwen/qwen3-30b-a3b-fp8"

type CFAIResult = {
  success: boolean
  result: {
    response?: string
    choices?: Array<{ message: { content: string } }>
  }
  errors: Array<{ code: number; message: string }>
}

export type AICallError = Error & {
  status: number
  cfErrors?: Array<{ code: number; message: string }>
}

export async function callAI(system: string, user: string): Promise<string> {
  const accountId = process.env.CLOUDFLARE_ACCOUNT_ID
  const apiToken = process.env.CLOUDFLARE_AI_TOKEN

  if (!accountId || !apiToken) {
    throw Object.assign(new Error("Workers AI credentials not configured"), { status: 500 })
  }

  const res = await fetch(
    `https://api.cloudflare.com/client/v4/accounts/${accountId}/ai/run/${MODEL}`,
    {
      method: "POST",
      headers: {
        Authorization: `Bearer ${apiToken}`,
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        messages: [
          { role: "system", content: system },
          { role: "user", content: user },
        ],
      }),
    }
  )

  let data: CFAIResult
  try {
    data = await res.json() as CFAIResult
  } catch {
    throw Object.assign(new Error("Workers AI non-JSON response"), { status: 500 })
  }

  if (!res.ok || !data.success) {
    const err = Object.assign(
      new Error(data.errors?.[0]?.message ?? `Workers AI error ${res.status}`),
      { status: res.status, cfErrors: data.errors }
    ) as AICallError
    throw err
  }

  const text = data.result.choices?.[0]?.message?.content ?? data.result.response ?? ""
  if (!text) throw Object.assign(new Error("Empty AI response"), { status: 500 })
  return text.trim()
}
```

---

## Migrating from REST API to AI Binding

1. Add `"ai": { "binding": "AI" }` to `wrangler.jsonc`
2. Replace the `fetch(api.cloudflare.com/...)` call with `env.AI.run(model, { messages })`
3. Get `env` via `getCloudflareContext()` (Next.js) or the Worker `fetch(req, env)` handler
4. Remove `CLOUDFLARE_ACCOUNT_ID` from `vars` in wrangler.jsonc
5. Delete the `CLOUDFLARE_AI_TOKEN` secret: `wrangler secret delete CLOUDFLARE_AI_TOKEN`
6. Test locally with `wrangler dev --remote`
