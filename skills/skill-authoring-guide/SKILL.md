---
name: "skill-authoring-guide"
description: "Guide for writing high-quality agent skills. Use when creating a new skill, editing an existing skill, reviewing skill quality, or when someone says 'how do I write a skill', 'skill structure', 'skill template', 'create a skill', 'skill best practices'. Also triggers when working with .agents/, .kimi/skills/, or SKILL.md files."
---

# Skill Authoring Guide

Comment écrire des skills de qualité pour l'écosystème Kimi.

---

## Qu'est-ce qu'un skill ?

Un skill = un fichier texte (`SKILL.md`) qui encode un **processus métier** ou une **expertise domaine**. C'est comme une fiche de procédure que l'agent consulte avant de répondre.

**Skill = "Comment faire"** (procédural, workflow)
**Agent = "Qui fait"** (persona, comportement)

---

## Structure obligatoire

```markdown
---
name: "skill-name"
description: "Ce que fait le skill + quand le déclencher. Soyez pushy."
---

# Skill Name

## Purpose
1 phrase sur ce que le skill fait et pourquoi.

## When to use
- Liste des situations qui déclenchent ce skill
- Liste des situations où NE PAS utiliser ce skill (boundary conditions)

## Workflow / Steps
1. Étape 1
2. Étape 2
3. Étape 3

## Anti-patterns / Don'ts
- Ce qu'il ne faut PAS faire
- Pièges courants

## References (optionnel)
- Fichiers externes à charger conditionnellement
```

---

## Règle #1 : La Description est le Trigger

**C'est la partie la plus importante du skill.** L'agent lit `name + description` pour décider s'il utilise le skill. Si la description est mauvaise, le skill ne sert à jamais.

### ✅ Bonne description

```yaml
description: "When the user wants to plan, design, or implement an A/B test or experiment, or analyze results from a completed test. Use when the user mentions 'split test', 'experiment', 'conversion rate optimization', 'test variants', 'statistical significance', or 'control group'. Do NOT use for general marketing strategy without testing component."
```

Pourquoi c'est bon :
- Dit CE QUE le skill fait (plan/design/implement A/B test)
- Dit QUAND le déclencher (liste de mots-clés)
- Dit QUAND NE PAS le déclencher (boundary condition)
- Est "pushy" — l'agent n'hésite pas à l'utiliser

### ❌ Mauvaise description

```yaml
description: "A skill for testing things"
```

Pourquoi c'est mauvais :
- Trop vague
- Pas de mots-clés de trigger
- Pas de boundary conditions
- L'agent ne saura jamais quand l'utiliser

### Template de description

```yaml
description: "[Ce que le skill fait]. Use when [situation A], [situation B], [situation C]. Also triggers on [keyword1], [keyword2], [keyword3]. Do NOT use for [situation où un autre skill est plus pertinent]."
```

---

## Règle #2 : Why-First

**Explique le POURQUOI avant le QUOI.** L'agent raisonne mieux quand il comprend les principes.

### ❌ Mauvais (règle sans explication)

```markdown
ALWAYS use pdfplumber for table extraction. NEVER use PyPDF2 for tables.
```

### ✅ Bon (règle avec raison)

```markdown
Use pdfplumber for table extraction. PyPDF2 is text-extraction focused and cannot preserve row/column structure. pdfplumber recognizes cell boundaries and returns structured data.
```

---

## Règle #3 : Progressive Disclosure

**Ne charge pas tout d'un coup.** Structure le skill pour que l'agent ne lise que ce dont il a besoin.

### Pattern 1 : Domaine spécifique

```
skill-name/
├── SKILL.md (overview + guide de sélection)
└── references/
    ├── finance.md (chargé si requête finance)
    ├── sales.md (chargé si requête sales)
    └── product.md (chargé si requête product)
```

### Pattern 2 : Conditionnel

```markdown
# API Design

## Simple CRUD
Use REST conventions. → See references/rest-basics.md

## Complex workflows with state transitions
Use state machines. → See references/state-machines.md
```

### Pattern 3 : Références volumineuses

Si un fichier de référence dépasse 300 lignes, ajouter un sommaire en haut :

```markdown
# API Reference

## Table of Contents
1. [Authentication](#auth)
2. [Endpoints](#endpoints)
3. [Error Codes](#errors)
```

---

## Règle #4 : Exemples > Explications

Un exemple concret vaut mieux qu'un paragraphe abstrait.

### ❌ Mauvais

```markdown
Write commit messages in conventional format.
```

### ✅ Bon

```markdown
## Commit Message Format

**Example 1:**
Input: Add JWT-based user authentication
Output: feat(auth): implement JWT-based authentication

**Example 2:**
Input: Fix password visibility toggle not working on login page
Output: fix(login): password visibility toggle button
```

---

## Règle #5 : Scripts Bundling

**Si un helper script est recréé à chaque session, le bundler dans le skill.**

### Comment détecter

Lire les transcripts de session. Si tu vois :
- "3 tests sur 3 créent le même helper script"
- "À chaque fois, le même pip install"
- "Même approche multi-étapes répétée"

→ C'est un candidat au bundling.

### Où le mettre

```
skill-name/
├── SKILL.md
└── scripts/
    └── helper.py (testé, validé, prêt à l'emploi)
```

---

## Règle #6 : Skill Reuse

**Avant de créer un nouveau skill, vérifier si un skill existant ne fait pas déjà le job.**

| Situation | Action |
|-----------|--------|
| Skill existant couvre 100% | Ne pas créer. Réutiliser. |
| Skill existant couvre 80% | Généraliser le skill existant |
| Domaine différent mais chevauchement | Créer un nouveau skill spécifique |
| Aucun overlap | Créer |

**Risque du skill bloat :** 50 skills qui se chevauchent = l'agent choisit le mauvais.

---

## Règle #7 : Test Before Deploy

**Tout nouveau skill DOIT passer par `skill-tester` avant d'être mergé.**

```bash
# Workflow recommandé
1. Écrire le skill dans ~/.kimi/skills-draft/{skill-name}/
2. Lancer skill-tester
3. Si APPROVED → merger dans ~/.kimi/skills/{skill-name}/
4. Si REJECTED → corriger ou abandonner
```

---

## Ce qu'il ne faut PAS mettre dans un skill

| ❌ Ne pas inclure | Pourquoi |
|-------------------|----------|
| README.md, CHANGELOG, INSTALLATION | Ce sont des docs humains, pas des instructions pour l'agent |
| Info de debugging du skill | Le skill est pour l'agent, pas pour le développeur du skill |
| Connaissances générales déjà connues | L'agent sait déjà écrire du Python. Ne pas lui apprendre. |
| Règles contradictoires | Une contradiction = l'agent ignore tout |

---

## Checklist avant publication

- [ ] Description contient trigger keywords + boundary conditions
- [ ] Au moins 3 étapes de workflow claires
- [ ] Au moins 1 exemple concret
- [ ] Anti-patterns listés
- [ ] Testé avec skill-tester (pass rate ≥ 70%)
- [ ] Pas de doublon avec un skill existant
- [ ] Progressive disclosure si le skill est > 200 lignes
