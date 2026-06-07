---
name: "skill-tester"
description: "Test and validate the quality of any skill before deployment. Run with-skill vs without-skill comparisons, assertion-based grading, and pass/fail evaluation. Use when creating a new skill, modifying an existing skill, or when output quality degrades. Trigger: 'test this skill', 'validate skill', 'skill quality check', 'compare with/without skill'."
---

# Skill Tester

Framework de validation des skills. Chaque skill doit prouver qu'il améliore la qualité par rapport à l'absence de skill.

## Quand utiliser

- Avant de merger un nouveau skill dans `~/.kimi/skills/`
- Après modification d'un skill existant
- Quand la qualité des outputs baisse (régression)
- Pour comparer 2 versions d'un même skill

## Principe fondamental

> Un skill qui ne sert à rien est pire que pas de skill. Il consomme du contexte sans apporter de valeur.

---

## Workflow de test (3 phases)

### Phase 1 : Préparation

Pour le skill à tester, définir :

```markdown
1. Nom du skill : ________________
2. Dossier : ~/.kimi/skills/________/
3. Date du test : ________________
4. Objectif du skill : 1 phrase
```

### Phase 2 : Test Cases (3 cas minimum)

Créer 3 prompts de test :

| # | Type | Prompt | Pourquoi |
|---|------|--------|----------|
| 1 | **Core** | La tâche la plus fréquente pour ce skill | Vérifie le cas nominal |
| 2 | **Edge** | Cas limite, ambigu, ou exceptionnel | Vérifie la robustesse |
| 3 | **Complex** | Tâche multi-étapes ou composite | Vérifie l'orchestration |

**Règles pour les prompts :**
- Utiliser des phrases naturelles (comme un vrai utilisateur)
- Inclure des détails concrets (noms de fichiers, contexte réel)
- Varier le ton (formel, casual, technique)

### Phase 3 : Exécution comparée

Pour chaque prompt, exécuter **2 fois** :

**Run A — With Skill :**
```
1. Charger le skill testé
2. Exécuter le prompt
3. Sauvegarder l'output dans :
   _workspace/skill-tests/{skill-name}/{date}/with-skill/case-{N}/
4. Noter le nombre de tokens utilisés
5. Noter le temps de réponse (approximatif)
```

**Run B — Without Skill (Baseline) :**
```
1. Désactiver temporairement le skill (renommer le dossier en .bak)
2. Exécuter le même prompt
3. Sauvegarder l'output dans :
   _workspace/skill-tests/{skill-name}/{date}/without-skill/case-{N}/
4. Noter le nombre de tokens utilisés
5. Noter le temps de réponse
6. Réactiver le skill
```

---

## Phase 4 : Grading (Assertion-based)

Pour chaque case, définir 3-5 assertions :

```json
{
  "skill_name": "pricing-strategist",
  "case_id": 1,
  "case_type": "core",
  "prompt": "Propose un pricing pour mon SaaS B2B",
  "assertions": [
    {
      "text": "Le output propose un modèle de pricing (et pas juste un prix)",
      "weight": 3
    },
    {
      "text": "Le output mentionne au moins 2 modèles possibles (ex: seat-based, usage-based)",
      "weight": 2
    },
    {
      "text": "Le output donne une fourchette de prix, pas un nombre unique",
      "weight": 2
    },
    {
      "text": "Le output mentionne les trade-offs entre modèles",
      "weight": 1
    }
  ]
}
```

**Règles d'assertion :**
- Chaque assertion doit être binaire (vrai/faux, pas subjectif)
- Le poids total doit faire 10
- Au moins 1 assertion doit tester ce que le skill apporte **spécifiquement** (vs baseline)

**Évaluation :**
```
Pour chaque case :
  Score_with = (somme des poids des assertions passées) / 10 × 100
  Score_without = même calcul sur le baseline
  Delta = Score_with - Score_without
```

---

## Phase 5 : Décision

### Critères de passage

| Métrique | Seuil | Action si échec |
|----------|-------|-----------------|
| **Delta moyen** | ≥ +30% | Rejeter si < +30% |
| **Score with-skill** | ≥ 70% | Rejeter si < 70% |
| **Tokens overhead** | ≤ +50% vs baseline | Optimiser si > +50% |
| **Tous les cases passent** | Score_with ≥ 60% chacun | Corriger les cases faibles |

### Verdict

```
✅ APPROVED — Le skill améliore significativement la qualité
⚠️ CONDITIONAL — Le skill améliore mais avec un coût trop élevé. Optimiser.
❌ REJECTED — Le skill n'apporte pas de valeur suffisante
```

**En cas de REJECTED :**
1. Analyser quelles assertions ont échoué
2. Identifier si le problème est :
   - Description trop vague (ne trigger pas au bon moment)
   - Instructions confuses (l'agent ne suit pas)
   - Overlap avec un skill existant
   - Skill inutile (l'agent fait déjà bien sans)
3. Soit corriger et retester, soit abandonner

---

## Description Trigger Test (Bonus)

Vérifier que la description du skill trigger au bon moment :

**Should-trigger (5 prompts) :**
- Variations de la requête principale
- Formulations différentes mais même intention
- Contexte implicite (pas de mot-clé exact)

**Should-NOT-trigger (5 prompts) :**
- Requêtes proches mais d'un autre domaine
- Requêtes que l'agent peut gérer sans skill
- Requêtes ambiguës où un autre skill est plus pertinent

**Si plus de 2 should-NOT-trigger se trompent → ajuster la description.**

---

## Structure du workspace de test

```
_workspace/
└── skill-tests/
    └── {skill-name}/
        └── {YYYY-MM-DD}/
            ├── with-skill/
            │   ├── case-1-core/
            │   │   ├── output.md
            │   │   ├── grading.json
            │   │   └── timing.json
            │   ├── case-2-edge/
            │   └── case-3-complex/
            ├── without-skill/
            │   ├── case-1-core/
            │   ├── case-2-edge/
            │   └── case-3-complex/
            ├── summary.json
            └── verdict.md
```

### summary.json

```json
{
  "skill_name": "pricing-strategist",
  "test_date": "2026-06-04",
  "results": [
    {
      "case": 1,
      "type": "core",
      "score_with": 90,
      "score_without": 50,
      "delta": 40,
      "tokens_with": 4500,
      "tokens_without": 2800,
      "tokens_overhead_pct": 61
    }
  ],
  "delta_average": 35,
  "score_with_average": 83,
  "tokens_overhead_average": 45,
  "verdict": "APPROVED",
  "recommendations": []
}
```

---

## Anti-patterns

- **Tester avec des prompts artificiels** → Toujours utiliser des phrases naturelles
- **N=1** → Minimum 3 cases, idéalement 5
- **Assertions subjectives** → "Bien écrit" est subjectif. "Contient une fourchette de prix" est objectif.
- **Ignorer le baseline** → Sans baseline, on ne sait pas si le skill sert à quelque chose
- **Tester après le merge** → Toujours tester AVANT de merger dans ~/.kimi/skills/
