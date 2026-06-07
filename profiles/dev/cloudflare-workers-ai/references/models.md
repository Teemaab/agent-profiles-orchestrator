# Model Selection

Full catalog: https://developers.cloudflare.com/workers-ai/models/

## Decision Tree

```
What do I need?
  ├─ Structured JSON output (strict schema)   → qwen3-30b-a3b-fp8
  ├─ General text, summaries, chat            → llama-3.3-70b (quality) or llama-3.1-8b (cost)
  ├─ Minimum neuron cost                      → llama-3.1-8b-instruct
  ├─ Best overall quality                     → llama-3.3-70b-instruct-fp8-fast
  └─ NEVER on free tier                       → kimi-k2.6 (kills 10k quota in 1-2 calls)
```

## Text Generation Models

| Model ID | Neurons / 1M output | Quality | JSON | Speed | Use when |
|----------|-------------------|---------|------|-------|----------|
| `@cf/qwen/qwen3-30b-a3b-fp8` | ~1,400 | High | Excellent | Fast | JSON output required, structured data |
| `@cf/meta/llama-3.3-70b-instruct-fp8-fast` | ~880 | Very high | Good | Medium | Best quality on a budget |
| `@cf/meta/llama-3.1-8b-instruct` | ~300 | Medium | Fair | Very fast | High volume, low cost, simple tasks |
| `@cf/moonshotai/kimi-k2.6` | ~363,000 | High | Good | Slow | **Free tier: never use.** Paid only. |

## Why Qwen for JSON

Qwen3-30b is a Mixture-of-Experts model: 30B total parameters, only 3B active per token. This gives it strong instruction-following at low compute cost. It reliably follows JSON schema instructions and rarely breaks out of the requested format when the prompt is well-structured.

Llama models can produce JSON but are more likely to add markdown fences (` ```json `) or trailing text that breaks `JSON.parse()`.

## Neuron Cost Explained

Neurons are Cloudflare's compute unit. They scale with **output tokens** (input tokens are cheap).

Approximate cost for a typical app call (300 output tokens):
| Model | Neurons per call |
|-------|----------------|
| llama-3.1-8b | ~0.09 |
| qwen3-30b-a3b-fp8 | ~0.42 |
| llama-3.3-70b | ~0.26 |
| kimi-k2.6 | ~109 |

On the free tier (10,000 neurons/day):
| Model | Max calls/day |
|-------|--------------|
| llama-3.1-8b | ~111,000 |
| qwen3-30b-a3b-fp8 | ~23,800 |
| llama-3.3-70b | ~38,400 |
| kimi-k2.6 | ~92 |

## Changing the Model

One-line change. Keep the model string as a constant:

```ts
// Binding
const MODEL = "@cf/qwen/qwen3-30b-a3b-fp8"
const result = await env.AI.run(MODEL, { messages })

// REST API
const res = await fetch(
  `https://api.cloudflare.com/client/v4/accounts/${accountId}/ai/run/${MODEL}`,
  ...
)
```
