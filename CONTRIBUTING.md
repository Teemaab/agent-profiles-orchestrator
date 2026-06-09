# Contributing a Skill

This guide covers everything needed to add a new skill to the orchestrator: file structure, SKILL.md checklist, eval format, and the review process.

---

## Workflow

```
draft SKILL.md  →  add evals  →  run skill-tester  →  open PR  →  merge
```

1. **Draft** — Create your skill directory under the appropriate profile and write `SKILL.md`.
2. **Evals** — Add at least 3 evals in `evals/evals.json` before opening a PR.
3. **Test** — Run the skill-tester against your evals and confirm all assertions pass.
4. **PR** — Open a pull request. All checklist items below must be satisfied.

---

## File Structure

Each skill lives in a profile directory:

```
profiles/<profile>/<skill-name>/
├── SKILL.md          # Agent instructions (required)
└── evals/
    └── evals.json    # Evaluation test cases (required)
```

Optional reference files loaded by the skill at runtime:

```
profiles/<profile>/<skill-name>/
├── references/       # Domain reference docs (loaded by SKILL.md)
│   └── *.md
└── scripts/          # Helper scripts referenced in SKILL.md
    └── *.py / *.sh
```

---

## SKILL.md Checklist

Every `SKILL.md` must include:

- [ ] **Frontmatter** — YAML block with all required fields (see below)
- [ ] **Trigger description** — `description:` field explains exactly when to use this skill and when NOT to
- [ ] **Workflow** — step-by-step instructions the agent follows
- [ ] **Output format** — what the agent produces (structure, labels, sections)
- [ ] **No tool-specific branding** — write for any agent (Claude, Cursor, Copilot); never name a specific AI product in instructions

### Required frontmatter

```yaml
---
name: <kebab-case-skill-name>
description: >
  One or two sentences: what this skill does, when to use it, and when NOT to use it.
  The orchestrator uses this field for routing decisions.
metadata:
  version: "1.0.0"
  author: "<your-github-handle>"       # créateur original
  # maintainer: "<github-handle>"      # optionnel : mainteneur actuel si différent de l'auteur
  # source: "community"                # optionnel : "community" | "core" | "vendor"
---
```

---

## Evals Format

File: `evals/evals.json`

```json
{
  "skill_name": "<same as name: in frontmatter>",
  "evals": [
    {
      "id": 1,
      "prompt": "The exact user message the skill should handle.",
      "expected_output": "A plain-text description of what a correct response looks like. Be specific about structure, findings, and what must NOT appear.",
      "assertions": [
        "Short, independently verifiable statement about the output",
        "Each assertion tests one thing",
        "Use positive phrasing: 'Output includes X', not 'Output does not lack X'"
      ],
      "files": []
    }
  ]
}
```

### Eval guidelines

- **Minimum 3 evals** per skill — cover happy path, edge case, and a negative case (something the skill should refuse or flag).
- **`prompt`** — write it as a real user message, not a test description.
- **`expected_output`** — describe the correct response in plain English. Include what must be present AND what must not appear.
- **`assertions`** — each item is a single, testable claim. Aim for 4–8 per eval.
- **`files`** — leave as `[]` unless the eval requires an attached file context.

---

## EXPERTS.md Update

If your new skill belongs to a new expert area, update the profile's `EXPERTS.md`:

```markdown
## Expert : <ExpertName>
**Trigger :** keyword1, keyword2, keyword3

| Skill | Dossier |
|-------|---------|
| your-skill-name | `your-skill-name/` |
```

If your skill fits an existing expert, add a row to the matching table in `EXPERTS.md`.

---

## PR Requirements

Before opening a PR, confirm all of the following:

- [ ] `SKILL.md` passes the frontmatter checklist above
- [ ] `evals/evals.json` exists with at least 3 evals
- [ ] All `assertions` pass when the skill is run against the eval prompts
- [ ] `EXPERTS.md` is updated if the routing map changed
- [ ] `INVENTORY.md` at repo root is updated with a new row (Skill | Profil | Expert | Tags)
- [ ] No references to non-existent skills, scripts, or files in `SKILL.md`
- [ ] `metadata.author` is set to your GitHub handle

---

## Skill Naming Conventions

| Type | Convention | Example |
|------|-----------|---------|
| Skill directory | `kebab-case` | `code-review-and-quality` |
| `name:` in frontmatter | `kebab-case` | `code-review-and-quality` |
| Profile directory | single word | `dev`, `marketing`, `security` |
| Eval `skill_name` | matches frontmatter `name:` | `code-review-and-quality` |

---

## Questions

Open an issue or check existing skills in `profiles/` for examples. The most complete references are:

- `profiles/dev/code-review-and-quality/` — typical dev skill pattern
- `profiles/marketing/copywriting/` — includes evals
- `profiles/architecture/tactical-ddd/` — multi-phase workflow pattern
