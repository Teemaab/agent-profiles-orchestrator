# Prompting & Optimization

## Prompting for JSON Output

Workers AI models do not have a native "JSON mode" toggle (unlike OpenAI). You must enforce JSON output through the prompt itself.

### System prompt structure that works

```
You are [role].

Produce a single JSON object matching this schema exactly:
[paste the schema with example values]

RULES:
- Return a single valid JSON object. No text outside the JSON.
- No asterisks, underscores, backticks, or any markdown.
- [domain-specific rules]
```

### Key rules to always include

```
- Return a single valid JSON object. No text outside the JSON.
- Plain text only. No asterisks, underscores, backticks, or any markdown.
```

Without these, Qwen and Llama will often wrap output in ` ```json ``` ` fences or add a preamble sentence. Both break `JSON.parse()`.

### Parsing with fallback stripping

Even with the rules, models occasionally add fences. Parse defensively:

```ts
function parseAIJson<T>(raw: string): T {
  // Strip markdown fences if present
  const cleaned = raw
    .replace(/^```(?:json)?\s*/i, "")
    .replace(/\s*```\s*$/, "")
    .trim()

  return JSON.parse(cleaned) as T
}
```

### Schema in the prompt

Paste a concrete example, not a TypeScript type. Models follow examples better than abstract schemas:

```
// Good -- concrete example with realistic values
{
  "type": "single_choice",
  "question": "What is your main constraint?",
  "options": ["Budget", "Time", "Technical skills"]
}

// Avoid -- abstract type definition
{ type: string, question: string, options: string[] }
```

---

## Optimizing for Neurons (Cost)

### 1. Keep output short

Neurons scale with output tokens. The shorter the required output, the cheaper the call.

- Use `"One sentence."` instead of `"Explain in detail."` in your schema instructions
- For lists, specify the exact count: `"Exactly 3 items."` prevents the model from generating 10
- For structured data, use short field names in the schema example

### 2. Keep the system prompt focused

A 4,000-token system prompt costs more than a 400-token one. Include only what the model needs for the current call:
- Remove examples that don't apply to the current page/task type
- Don't repeat rules that are implicit (e.g., don't say "respond in JSON" five times)

### 3. Separate heavy and light calls

If your app makes multiple AI calls per user action, check if some can be:
- **Merged**: combine two small prompts into one structured output with two sections
- **Cached**: if the output is deterministic given the same input, cache it in KV or the database
- **Deferred**: generate on-demand (user clicks "Generate X") instead of upfront

### 4. Choose the right model for the task

| Task | Model |
|------|-------|
| Structured JSON with complex schema | qwen3-30b-a3b-fp8 |
| Simple classification or short answer | llama-3.1-8b-instruct |
| High-stakes text where quality matters | llama-3.3-70b-instruct-fp8-fast |

---

## Common Failures and Fixes

| Symptom | Cause | Fix |
|---------|-------|-----|
| `JSON.parse()` throws | Model added markdown fences | Add stripping fallback + `"No text outside the JSON"` rule |
| Field is `"N/A"` or `"non specifie"` | Prompt said "don't invent" without alternative | Change to "always estimate from industry knowledge, never output N/A" |
| Response cuts off mid-JSON | Output token limit reached | Shorten the schema, reduce list sizes, or split into multiple calls |
| Model ignores a rule | Rule buried in a long prompt | Move critical rules to the end of the system prompt, use `CRITICAL:` prefix |
| Output in wrong language | No language instruction | Add `"Respond in French."` / `"Respond in the same language as the conversation."` |

---

## Conversation History Format

For multi-turn conversations, pass history as a formatted string in the user message rather than as multiple message objects. Workers AI models handle this reliably:

```ts
function formatHistory(
  turns: Array<{ role: string; content: unknown }>
): string {
  return turns
    .map((t) => `${t.role.toUpperCase()}: ${JSON.stringify(t.content)}`)
    .join("\n")
}

// Usage
const user = `## Conversation History\n${formatHistory(history)}\n\nRespond with JSON only.`
```

This keeps the system prompt clean and gives the model the full context in a predictable format.
