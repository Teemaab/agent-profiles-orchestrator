# Architecture detaillee

## Phase 1 : Router

Le router classifie la requete sans intervention humaine.

| Mots-cles | Profil |
|-----------|--------|
| code, API, bug, refactor, deploy, test, component | **dev** |
| pricing, revenue, strategy, board, contract, fundraise | **business** |
| SEO, copy, ads, launch, social, campaign, content | **marketing** |
| audit, pentest, vulnerability, threat, compliance | **security** |
| architecture, DDD, RFC, pattern, microservice | **architecture** |
| UI, UX, Figma, design system, visual | **design** |

Si ambigu : l'agent demande "Tu veux que je traite ca en marketing, dev, ou les deux ?"

## Phase 2 : Profile Loader

1. Confirmer : "Profil [X] active."
2. Charger la memoire : `memory.md`
3. Charger les experts : `EXPERTS.md`
4. Initialiser le contexte du profil

## Phase 3 : Supervisor

| Niveau | Mots-cles | Workflow | Skills max |
|--------|-----------|----------|------------|
| Low | "explique", "resum", "liste", "avis", "ide" | Expert Pool simple | 1 |
| Medium | "comment faire", "conseil", "ameliorer" | Expert Pool standard | 3 |
| High | "code prod", "feature", "API", "deployer", "pricing", "strategie" | Producer-Reviewer | 5 |
| Critical | "securite", "auth", "paiement", "RGPD", "contrat", "architecture", "levee" | Producer-Reviewer + Validation | 7 |

## Phase 4 : Expert Pool

Evalue la pertinence de chaque expert face a la requete :

| Pertinence | Action |
|------------|--------|
| **Tres pertinent** | L'expert correspond parfaitement. Utilise directement. |
| **Pertinent** | L'expert est plausible. Cross-consultation + proposition alternative. |
| **Secondaire** | Aucun expert ne correspond bien. Broad Search Fallback. |

## Phase 5 : Producer-Reviewer (High/Critical)

**Producer** : execute le skill, produit l'output.

**Reviewer** verifie selon le domaine :
- **Code** : compile-t-il ? Injections SQL/XSS ? Tests passent ?
- **Business** : chiffres coherents ? Risques mentionnes ?
- **Marketing** : copy clair ? CTA present ? Preuve sociale ?

Max 2 iterations (eviter boucle infinie).

## Phase 6 : Sanity Check (obligatoire)

Avant d'envoyer la reponse :

1. **Cohérence sujet** : la requete correspond-elle au projet dans `memory.md` ?
2. **Cohérence chiffres** : les prix/metriques cites existent-ils dans `memory.md` ?
3. **Cohérence noms** : le nom du produit dans la reponse correspond-il au `memory.md` ?
4. **Cohérence audience** : la cible mentionnee correspond-elle a l'audience du projet ?
5. **Context Null Check** : si la requete ne mentionne aucun projet connu, demander "De quel projet parles-tu ?"

En cas d'alerte : STOP, identifier le bon contexte, recharger le bon `memory.md`, re-executer.

> Pas de pourcentages. Pas de scores. Juste des verifications binaires : OK ou STOP.

## Phase 7 : Skills Loader

1. Lire le SKILL.md du skill selectionne
2. Si cross-profile : lire aussi les skills secondaires
3. Executer le Sanity Check
4. Synthetiser la reponse
5. Mettre a jour `memory.md` si nouveau contexte

## Structure d'un skill

```
skills/mon-skill/
├── SKILL.md              # Definition (YAML frontmatter + Markdown)
├── references/
│   ├── standards.md      # Mappings MITRE, NIST, OWASP
│   └── workflows.md      # Procedures detaillees
├── scripts/
│   └── process.py        # Scripts helpers
└── assets/
    └── template.md       # Templates et checklists
```

### YAML frontmatter

```yaml
---
name: mon-skill-name
description: >-
  Ce que fait ce skill. Mots-cles riches pour la decouverte.
domain: cybersecurity
subdomain: web-security
tags: [web, owasp, injection, api]
version: "1.0"
author: ton-nom
license: Apache-2.0
---
```

## Structure d'un profil

```
profiles/dev/
├── memory.md             # Contexte persistant du projet
├── EXPERTS.md            # Mapping experts -> skills
└── [skills specifiques]  # Skills propres au profil
```

### memory.md

Contient :
- Nom du projet et mission
- Stack technique
- Conventions de code
- Regles metier critiques (non-negociables)
- Pricing et metriques
- Architecture patterns adoptes

### EXPERTS.md

Contient :
- Liste des experts du profil
- Mots-cles declencheurs par expert
- Mapping expert -> skills physiques
