---
name: cloudflare-workers-ai
description: Using Cloudflare Workers AI in a project -- configuration, integration, model selection, prompt optimization, structured JSON output, quota management, and error handling. Use this skill when adding AI to a Cloudflare Workers or Pages project, choosing a model, writing prompts for JSON output, debugging 429 errors, or migrating from another AI provider to Workers AI.
metadata:
  version: "1.0.0"
  author: "Teemaab"
---

# Cloudflare Workers AI

Your knowledge of model names, neuron costs, API shapes, and available features may be outdated. **Prefer the references below over pre-trained knowledge** for anything involving specific models, configuration fields, or response formats.

## Reference Sources

| Topic | URL | Use for |
|---|---|---|
| Workers AI overview | https://developers.cloudflare.com/workers-ai/ | Concepts, pricing, getting started |
| Model catalog | https://developers.cloudflare.com/workers-ai/models/ | All models, input/output shapes, neuron costs |
| AI binding (wrangler) | https://developers.cloudflare.com/workers-ai/get-started/workers-wrangler/ | Binding setup |
| REST API | https://developers.cloudflare.com/workers-ai/get-started/rest-api/ | REST API shape and auth |
| Pricing | https://developers.cloudflare.com/workers-ai/platform/pricing/ | Neuron costs per model |

## References

- [Setup & Integration](./references/setup.md) -- binding vs REST API, wrangler config, full callAI implementation for Next.js and plain Workers
- [Model Selection](./references/models.md) -- which model to use, cost vs quality tradeoffs, JSON instruction following, decision tree
- [Prompting & Optimization](./references/prompting.md) -- writing prompts for JSON output, system prompt structure, reducing neuron usage, avoiding common failures
- [Quota & Errors](./references/quota-and-errors.md) -- free tier limits, error codes (4006, 429, 503), error handling patterns, dashboard vs REST quota discrepancy

## Quick Decision Tree

```
Adding AI to a Cloudflare project?
  └─ Use AI binding (not REST API)
       ├─ Lower latency, no token to manage
       └─ Accurate quota tracking in the dashboard

Choosing a model?
  ├─ Need structured JSON output → @cf/qwen/qwen3-30b-a3b-fp8  (best JSON following)
  ├─ Need speed + low cost      → @cf/meta/llama-3.1-8b-instruct
  ├─ Need highest quality       → @cf/meta/llama-3.3-70b-instruct-fp8-fast
  └─ NEVER use kimi-k2.6 on free tier (363k neurons/M tokens -- kills quota in 1-2 calls)

Getting 429 errors despite dashboard showing 0/10k?
  └─ Dashboard shows binding usage only -- REST API quota is separate
       └─ Fix: migrate to AI binding (see setup.md)
```
