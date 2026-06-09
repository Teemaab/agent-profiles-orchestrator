---
name: stop-slop
description: Remove AI writing patterns from prose. Use when drafting, editing, or reviewing text to eliminate predictable AI tells.
metadata:
  trigger: Writing prose, editing drafts, reviewing content for AI patterns
  author: Hardik Pandya (https://hvpandya.com)
---

# Stop Slop

Eliminate predictable AI writing patterns from prose.

## Core Rules

1. **Cut filler phrases.** Remove throat-clearing openers, emphasis crutches, and all adverbs. See [references/phrases.md](references/phrases.md).

2. **Break formulaic structures.** Avoid binary contrasts, negative listings, dramatic fragmentation, rhetorical setups, false agency. See [references/structures.md](references/structures.md).

3. **Use active voice.** Every sentence needs a human subject doing something. No passive constructions. No inanimate objects performing human actions ("the complaint becomes a fix").

4. **Be specific.** No vague declaratives ("The reasons are structural"). Name the specific thing. No lazy extremes ("every," "always," "never") doing vague work.

5. **Put the reader in the room.** No narrator-from-a-distance voice. "You" beats "People." Specifics beat abstractions.

6. **Vary rhythm.** Mix sentence lengths. Avoid rhetorical three-item lists used as decoration. Technical enumerations are allowed. End paragraphs differently. No em dashes. No tirets typographiques.

7. **Trust readers.** State facts directly. Skip softening, justification, hand-holding.

8. **Cut quotables.** If it sounds like a pull-quote, rewrite it.

## Quick Checks

Before delivering prose:

- Any adverbs? Kill them.
- Any passive voice? Find the actor, make them the subject.
- Inanimate thing doing a human verb ("the decision emerges")? Name the person.
- Sentence starts with a Wh- word? Restructure it.
- Any "here's what/this/that" throat-clearing? Cut to the point.
- Any "not X, it's Y" contrasts? State Y directly.
- Three consecutive sentences match length? Break one.
- Paragraph ends with punchy one-liner? Vary it.
- Em-dash anywhere? Remove it. Replace with a comma, a period, a colon, or parentheses. Never use typographic dashes.
- Vague declarative ("The implications are significant")? Name the specific implication.
- Narrator-from-a-distance ("Nobody designed this")? Put the reader in the scene.
- Meta-joiners ("The rest of this essay...")? Delete. Let the essay move.

## French Writing

### Language Integrity

When writing in French, preserve all diacritics and accents.

- Use: é, è, ê, à, ù, ç, ô, ï, ë when required.
- Never replace with ASCII equivalents for convenience.
- Incorrect: securite, developpement, strategie
- Correct: sécurité, développement, stratégie

### French AI Patterns

Remove these French-language AI tells. They are the French equivalents of
"Here's the thing" and "It's worth noting." See [references/phrases.md](references/phrases.md).

### Punctuation (any language)

No em dashes (—) or en dashes (–) used as sentence separators.
Replace with: period, comma, colon, or parentheses.

Bad: The system works — when configured correctly.
Good: The system works when configured correctly.
Good: The system works. Configure it correctly.

---

## Confidence Calibration

Do not claim certainty the writer cannot demonstrate.

Avoid:
- guarantees / garantit
- ensures / assure
- prevents / empêche
- eliminates / élimine
- impossible
- always / toujours (as a universal claim)

Prefer:
- reduces / réduit
- improves / améliore
- helps / aide
- increases likelihood / augmente les chances
- in most cases / dans la plupart des cas

Bad: "This pattern prevents SQL injection."
Good: "This pattern reduces SQL injection risk."

Bad: "Our approach guarantees zero downtime."
Good: "Our approach targets zero downtime."

---

## Technical Writing Exception

This skill targets style, not substance. Do not remove:

- Technical terminology (HTTP methods, algorithm names, protocol names)
- Precise definitions and specifications
- RFC/security/architectural vocabulary
- Enumerated technical properties (even if there are three or more)

Prefer precision over minimalism. If removing a word changes the technical
meaning, keep the word.

---

## Scoring

Rate 1-10 on each dimension:

| Dimension | Question |
|-----------|----------|
| Directness | Statements or announcements? |
| Rhythm | Varied or metronomic? |
| Trust | Respects reader intelligence? |
| Authenticity | Sounds human? |
| Density | Anything cuttable? |

Below 35/50: revise.

## Examples

See [references/examples.md](references/examples.md) for before/after transformations.

## License

MIT
