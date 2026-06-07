---
name: design-anti-patterns
description: |
  Use when a design looks generic, AI-generated, bland, or overly busy.
  Use when auditing UI for common visual tells and cheap patterns.
  Use when a landing page, dashboard, or component feels off but the cause is unclear.
  Use when reviewing typography, color, spacing, motion, or copy for professionalism.
  Do NOT use for backend architecture or non-visual tasks.
metadata:
  author: fadi
  version: '1.0.0'
---

# Design Anti-Patterns

Detect and fix the visual tells that make designs look cheap, generic, or AI-generated.

## Color

### Gray Text on Colored Backgrounds
**BAD:** Muted gray (`#9CA3AF`) on a tinted background looks washed out and unreadable.
**GOOD:** Use a darker shade of the background's own hue, or a transparency of the text color.

### Pure Black on White
**BAD:** `#000000` on `#FFFFFF` creates harsh contrast and eye strain.
**GOOD:** Use tinted neutrals — near-black with a hue shift (`#0C0A09` on warm white, `#0A0A0F` on cool white).

### Gradient Overload
**BAD:** Cyan-to-purple gradients, glassmorphism, neon accents on dark backgrounds.
**GOOD:** One strategic accent color + tinted neutrals. Gradient only when it serves hierarchy.

### Low Contrast Body Text
**BAD:** Light gray body text (`text-gray-400`) on near-white for "elegance."
**GOOD:** Body text must hit ≥4.5:1 contrast. If close, bump toward the ink end of the ramp.

## Typography

### Oversized Hero Headlines
**BAD:** `clamp(3rem, 10vw, 11rem)` — 128-176px reads as shouting, not bold.
**GOOD:** Hero ceiling: `clamp()` max ≤ 6rem (~96px). Above that is comically loud.

### Tight Letter-Spacing on Display
**BAD:** `letter-spacing: -0.05em` to `-0.085em` on H1s makes letters touch = cramped.
**GOOD:** Display tracking floor: ≥ -0.04em. `-0.02em` to `-0.03em` is plenty for tight grotesque.

### All-Caps Body Copy
**BAD:** Sentences in ALL CAPS at body sizes = unreadable.
**GOOD:** Reserve uppercase for short labels (≤4 words), badges, and eyebrows used sparingly.

### Flat Type Scale
**BAD:** Same weight and size for H1, H2, H3 with no contrast.
**GOOD:** Hierarchy through scale + weight contrast (≥1.25 ratio between steps).

### Too Many Font Families
**BAD:** 4+ fonts competing (geometric sans + humanist sans + serif + mono).
**GOOD:** Max 3 families (display + body + optional mono). One well-tuned family with weight contrast beats three competing faces.

### Long Line Length
**BAD:** Body text spanning full width on desktop = 120ch+.
**GOOD:** Cap body line length at 65–75ch for readability.

## Layout

### Cards Inside Cards
**BAD:** Nested cards with identical padding, border-radius, and shadow.
**GOOD:** Cards are the lazy answer. Use them only when they're the best affordance. One level max.

### Monotone Spacing
**BAD:** Everything has `margin-bottom: 24px` or `gap: 16px` everywhere.
**GOOD:** Vary spacing for rhythm. Tighter within groups, looser between sections.

### Arbitrary Z-Index
**BAD:** `z-index: 999` or `z-index: 9999`.
**GOOD:** Build a semantic z-index scale: dropdown → sticky → modal-backdrop → modal → toast → tooltip.

### Defaulting to Grid
**BAD:** Using CSS Grid for simple 1D layouts where `flex-wrap` would be simpler.
**GOOD:** Flexbox for 1D, Grid for 2D.

## Motion

### Bounce and Elastic Easing
**BAD:** `ease-in-out-bounce`, `elastic` easing = feels dated and toy-like.
**GOOD:** Ease out with exponential curves (`ease-out-quart`, `ease-out-expo`).

### Image Hover Animation
**BAD:** `transform: scale(1.05)` on `<img>` hover via parent `.group:hover`.
**GOOD:** Animate the card's background, border, or shadow. Never the image itself.

### Gating Content on Animation
**BAD:** Content invisible by default, revealed only by a scroll-triggered class.
**GOOD:** Content visible by default. Animation enhances, never gates. Respect `prefers-reduced-motion`.

## Copy

### Em Dashes in Body Copy
**BAD:** `—` used repeatedly as a stylistic device in sentences.
**GOOD:** Use commas, colons, semicolons, or parentheses. Never `--` or `—` in visible text.

### Aphoristic Cadence
**BAD:** Recurring rhythm of "serious statement, then punchy short negation" across sections.
**GOOD:** Specific, not aphoristic. If three sections end on a short rebuttal-shaped sentence, rewrite.

### Restated Headings
**BAD:** Section copy that repeats the heading in different words.
**GOOD:** Every word earns its place. No intros that restate the title.

## Checklist

- [ ] No gray text on colored backgrounds
- [ ] No pure black on pure white
- [ ] Body contrast ≥4.5:1
- [ ] Hero headline ≤6rem
- [ ] Display tracking ≥-0.04em
- [ ] No all-caps body copy
- [ ] Type scale has ≥1.25 ratio between steps
- [ ] Max 3 font families
- [ ] Body line length ≤75ch
- [ ] No nested cards
- [ ] Varied spacing for rhythm
- [ ] Semantic z-index scale
- [ ] No bounce/elastic easing
- [ ] No image hover transforms
- [ ] Content visible without animation
- [ ] No em dashes in body copy
- [ ] No restated headings
