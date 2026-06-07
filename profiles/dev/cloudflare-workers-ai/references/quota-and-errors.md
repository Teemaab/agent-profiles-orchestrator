# Quota & Error Handling

## Free Tier Limits

| Plan | Neurons / day | Rate | Reset |
|------|--------------|------|-------|
| Free | 10,000 / day | Low | 00:00 UTC |
| Workers Paid ($5/mo) | 10,000 / minute | High | Per minute |

**Dashboard discrepancy:** The Workers AI dashboard tracks **binding** usage only. REST API calls are tracked separately and may exhaust the quota without updating the dashboard. If the dashboard shows `0/10k` but REST API calls return 429 (code 4006), migrate to the AI binding (see setup.md).

## Error Codes

| HTTP | CF Code | Meaning | User message |
|------|---------|---------|-------------|
| 429 | 4006 | Daily free quota exhausted | "Daily limit reached. Try again tomorrow." |
| 429 | other | Rate limit (req/min exceeded) | "Too many requests. Try again in X seconds." |
| 503 | - | Model temporarily overloaded | "Service unavailable. Try again in a moment." |
| 400 | - | Invalid model name or malformed request | Log and return generic error |
| 401 | - | Invalid or missing API token | Check `CLOUDFLARE_AI_TOKEN` secret |

## Error Handling Pattern (REST API)

```ts
// In errors.ts
import { NextResponse } from "next/server"
import type { AICallError } from "./client"

export function aiErrorResponse(error: unknown): NextResponse | null {
  const err = error as AICallError

  if (err.status === 429) {
    const isDaily = err.cfErrors?.some((e) => e.code === 4006)
    if (isDaily) {
      return NextResponse.json({ error: "daily_quota" }, { status: 429 })
    }
    return NextResponse.json(
      { error: "quota_exceeded", retryAfter: "60s" },
      { status: 429 }
    )
  }

  if (err.status === 503) {
    return NextResponse.json({ error: "service_unavailable" }, { status: 503 })
  }

  return null
}
```

```ts
// In apiError.ts (client-side message mapping)
export function apiErrorMessage(status: number, data: unknown): string | null {
  const d = (data ?? {}) as { error?: string; retryAfter?: string }

  if (status === 429) {
    return d.error === "daily_quota"
      ? "Daily limit reached. Try again tomorrow."
      : `Too many requests. Try again in ${d.retryAfter ?? "a few seconds"}.`
  }

  if (status === 503) {
    return "Service temporarily unavailable. Try again in a moment."
  }

  return null
}
```

## Error Handling Pattern (Binding)

With the AI binding, errors surface as thrown exceptions. Wrap the call:

```ts
try {
  const result = await env.AI.run(MODEL, { messages })
  // ...
} catch (err: unknown) {
  const e = err as { message?: string; status?: number }
  // Binding throws with a message like "daily free allocation" for quota errors
  if (e.message?.includes("daily free allocation")) {
    return NextResponse.json({ error: "daily_quota" }, { status: 429 })
  }
  if (e.message?.includes("overloaded") || e.status === 503) {
    return NextResponse.json({ error: "service_unavailable" }, { status: 503 })
  }
  return NextResponse.json({ error: "ai_error" }, { status: 500 })
}
```

## Checking Quota in the Dashboard

1. Go to `dash.cloudflare.com > Workers & Pages > Workers AI`
2. "Neurons used today" shows **binding** usage only
3. For REST API usage, check the API analytics (if available) or switch to binding for unified tracking
