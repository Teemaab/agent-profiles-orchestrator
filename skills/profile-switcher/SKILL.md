---
name: "profile-switcher"
description: "Orchestrate the full agent architecture: Router → Profile → Supervisor → Expert Pool → Producer-Reviewer → Skills. With safety guards: fallback broad search, cross-profile auto-trigger, and coverage audit. Handles cross-profile consultation, criticality levels, memory per profile, and expert pertinence evaluation."
---

# Profile Switcher — Architecture Orchestrator

## Architecture

```
User Request
    ↓
[ROUTER] — Classify into Profile
    ↓
[PROFILE] — Load memory + experts mapping
    ↓
[SUPERVISOR] — Decide workflow + criticality
    ↓
[EXPERT POOL] — Select expert(s) by pertinence
    ↓
  ├─ Tres pertinent → Proceed
  ├─ Pertinent → Cross-consult + propose alternative
  └─ Secondaire → Broad Search Fallback
    ↓
[PRODUCER-REVIEWER] — Quality gate (if High/Critical)
    ↓
[SKILLS] — Load specific skill(s)
    ↓
Response
```

---

## Phase 1 : Router

**Goal :** Déterminer le profil principal sans que l'utilisateur le précise.

### Auto-routing (sans "bascule vers")

Si l'utilisateur ne précise pas de profil, classifier la requête :

| Mots-clés | Profil |
|-----------|--------|
| code, API, bug, refactor, deploy, test, component | **dev** |
| pricing, revenue, strategy, board, contract, fundraise | **business** |
| SEO, copy, ads, launch, social, campaign, content | **marketing** |
| architecture, DDD, RFC, pattern, microservice | **architecture** |
| audit, pentest, vulnerability, threat, compliance | **security** |
| UI, UX, Figma, design system, visual | **design** |

**Règle :** Si ambigu (ex: "landing page" = marketing + dev), demander : *"Tu veux que je traite ça en marketing, dev, ou les deux ?"*

---

## Phase 2 : Profile Loader

**Actions :**
1. Confirmer : `"Profil [X] activé."`
2. Charger la mémoire : `ReadFile("~/.kimi/skills-profiles/{profil}/memory.md")`
3. Charger les experts : `ReadFile("~/.kimi/skills-profiles/{profil}/EXPERTS.md")`
4. Initialiser le contexte du profil

---

## Phase 3 : Supervisor

**Goal :** Décider du workflow et du niveau de criticité.

### Détection de criticité

| Niveau | Mots-clés déclencheurs | Workflow | Skills max |
|--------|------------------------|----------|------------|
| **Low** | "explique", "résume", "liste", "avis", "idée" | Expert Pool simple | 1 |
| **Medium** | "comment faire", "conseil", "améliorer" | Expert Pool standard | 3 |
| **High** | "code prod", "feature", "API", "déployer", "pricing", "stratégie" | Producer-Reviewer | 5 |
| **Critical** | "sécurité", "auth", "paiement", "RGPD", "contrat", "architecture", "levée" | Producer-Reviewer + Validation | 7 |

### Sélection du workflow

```
if Critical or High:
    workflow = "Producer-Reviewer"
elif Medium:
    workflow = "Expert-Pool"
elif Low:
    workflow = "Single-Skill"

# Exception : brainstorming, étude de marché, analyse comparative → Fan-Out
if "brainstorm" or "étude de marché" or "comparatif" or "analyse multi-angle":
    workflow = "Fan-Out"
```

### Cross-Profile Detection

Si la requête implique plusieurs domaines :

| Requête contient... | Profil principal | Profils secondaires |
|---------------------|-----------------|---------------------|
| SaaS + cybersécurité | business | security, dev |
| Landing page + SEO | marketing | dev, design |
| Architecture + perf | dev | architecture |
| Pricing + contrat | business | commercial-skills |
| Code + sécurité | dev | security |

**Action :** Noter les profils secondaires pour consultation ultérieure.

---

## Phase 4 : Expert Pool — AVEC GARDE-FOUS

**Goal :** Sélectionner le bon expert au sein du profil actif, **sans jamais rater un skill pertinent**.

### Étape 4a : Evaluation de la pertinence

Pour chaque expert dans EXPERTS.md, evaluer la correspondance avec la requete (pas de pourcentage, pas de score mathematique).

### Étape 4b : Decision selon la pertinence

| Pertinence | Action | Message utilisateur |
|------------|--------|---------------------|
| **Tres pertinent** | Correspondance parfaite — utiliser directement | (silencieux) |
| **Pertinent** | Correspondance plausible — **Cross-consultation** | "J'ai selectionne [Expert A]. Tu veux aussi consulter [Expert B] ?" |
| **Secondaire** | Aucun expert ne correspond bien — **Broad Search Fallback** | "Je ne suis pas sur du meilleur expert. Je scanne tous les domaines..." |

---

## 🛡️ Garde-fou 1 : Broad Search Fallback (pertinence Secondaire)

**Quand :** Aucun expert n'est "Tres pertinent" ou "Pertinent".

**Action :**
```markdown
1. Ne PAS choisir un expert au hasard
2. Scanner les descriptions de TOUS les experts du profil actif
3. Identifier les 3 meilleurs candidats (même si pertinence Secondaire)
4. Lire les skills des 3 experts
5. Synthétiser une réponse combinée
6. Prévenir l'utilisateur : "J'ai consulté plusieurs experts. Voici la synthèse."
```

**Pourquoi :** Évite le false negative. Mieux vaut trop consulter que rater le bon expert.

---

## 🛡️ Garde-fou 2 : Cross-Profile Auto-Trigger

**Quand :** La requête contient des mots-clés de plusieurs profils.

**Action automatique :**
```markdown
Si la requête contient des mots-clés de profils secondaires :
1. Charger les experts des profils secondaires
2. Evaluer la pertinence des experts secondaires
3. Si un expert secondaire est Pertinent ou Tres pertinent :
   - Consulter son skill
   - Intégrer sa perspective dans la réponse finale
4. Indiquer : "J'ai aussi consulté l'expert [X] du profil [Y] pour compléter."
```

**Exemple :**
> **Requête :** "J'ai une fuite mémoire dans mon auth service JWT"
> 
> **Router** → dev (principal)
> **Cross-Profile** → "auth" + "JWT" + "fuite mémoire" détecte aussi security + performance
> **Action** → Consulter Backend (dev) + Security (security) + Performance (dev)
> **Réponse** → Synthèse des 3 perspectives

---

## 🛡️ Garde-fou 3 : "Did you mean?" (pertinence Pertinent)

**Quand :** Le meilleur expert est Pertinent, mais un second expert est proche.

**Action :**
```markdown
1. Répondre avec le skill choisi (Expert A)
2. AJOUTER en fin de réponse :
   "J'ai utilisé l'expert [A]. Si tu préfères, je peux aussi consulter l'expert [B] qui semble pertinent."
3. Laisser l'utilisateur corriger
```

**Pourquoi :** Donne le contrôle à l'utilisateur sans le surcharger de choix.

---

## 🛡️ Garde-fou 4 : Skill Coverage Audit (mensuel)

**Quand :** Une fois par mois, ou sur demande.

**Action :**
```markdown
1. Lister tous les skills du profil
2. Compter combien ont été utilisés sur les 30 derniers jours
3. Identifier les skills "invisibles" (jamais triggerés)
4. Pour chaque skill invisible :
   - Vérifier si sa description est claire
   - Vérifier si son expert est bien classifié
   - Si le skill est utile mais jamais trouvé → corriger EXPERTS.md
   - Si le skill est inutile → marquer pour suppression
5. Générer un rapport : "Couverture : 73% | Skills orphelins : 12 | À corriger : 3"
```

**Metrique :** Skill Coverage = (skills utilises) / (skills total). Objectif : plus de 70% de couverture.

---

## Phase 5 : Producer-Reviewer (si High/Critical)

### Producer

1. Charger le skill sélectionné
2. Exécuter la requête
3. Produire l'output

### Reviewer

Vérifier l'output selon le domaine :

**Pour le code :**
- Le code compile-t-il ?
- Y a-t-il des injections SQL / XSS ?
- Les tests passent-ils ?

**Pour le business :**
- Les chiffres sont-ils cohérents ?
- Y a-t-il des oublis importants ?
- Les risques sont-ils mentionnés ?

**Pour le marketing :**
- Le copy est-il clair ?
- Le CTA est-il présent ?
- La preuve sociale est-elle incluse ?

**Action si review échoue :**
1. Lister les problèmes
2. Retourner au Producer avec corrections
3. Max 2 itérations (éviter boucle infinie)

---

## Phase 6 : Sanity Check (obligatoire)

**Goal :** Vérifier que la réponse générée correspond au bon contexte AVANT de l'envoyer.

Cette phase existe pour éviter les confusions de contexte (ex: répondre sur le projet A alors que l'utilisateur parle du projet B).

### Checklist Sanity Check

Avant de produire la réponse finale, vérifier :

**1. Cohérence sujet :**
- Le sujet de la requête correspond-il au projet décrit dans `memory.md` ?
- Exemples de mismatch :
  - memory.md parle de "e-commerce" mais la requête parle de "cours en ligne" → ALERTE
  - memory.md parle de "10 000 utilisateurs" mais la requête parle de "B2B enterprise" → ALERTE

**2. Cohérence chiffres :**
- Les prix mentionnés existent-ils dans `memory.md` ?
- Les métriques citées existent-elles dans `memory.md` ?
- Si un chiffre est nouveau (ex: correction utilisateur), le noter et mettre à jour memory.md

**3. Cohérence noms :**
- Le nom du produit dans la réponse correspond-il au nom dans `memory.md` ?
- Exemple : ne pas dire "Projet A" quand memory.md dit "Projet B"

**4. Cohérence audience :**
- La cible mentionnée dans la réponse correspond-elle à l'audience du projet ?
- Exemple : ne pas parler d'"élèves du BAC" pour un SaaS B2B (cible = professionnels 25-45 ans)

**5. Context Null Check (nouveau) :**
- Si la requête de l'utilisateur ne mentionne explicitement aucun projet connu (ni Projet A, ni Projet B, ni autre produit référencé dans `memory.md`) :
  - NE PAS injecter le contexte du dernier sujet traité par défaut
  - NE PAS répondre de manière générique en se basant sur `memory.md`
  - STOP et demander : *"De quel projet parles-tu ?"*
- Exemple : memory.md contient "Projet A", la requête est "comment faire un cours ?" → demander le projet cible, ne pas supposer "Projet A"

### En cas d'alerte

```markdown
Si un mismatch est détecté :
1. STOP — ne pas envoyer la réponse
2. Identifier le bon contexte (quel projet ? quel profil ?)
3. Recharger le bon memory.md
4. Re-exécuter l'expert pool avec le bon contexte
5. Si l'ambiguïté persiste : demander à l'utilisateur "Tu parles bien de [projet] ?"
```

### Mise à jour de mémoire

Si l'utilisateur fournit une correction (ex: "L'offre fondateur c'est 40€, pas 19€") :
1. Corriger immédiatement `memory.md`
2. Inclure la source : "Mis à jour suite à correction utilisateur"
3. Ne plus répéter l'erreur

---

## Phase 7 : Skills Loader

**Actions finales :**
1. Lire le SKILL.md du skill sélectionné
2. Si cross-profile : lire aussi les skills des profils secondaires
3. Exécuter le Sanity Check
4. Synthétiser la réponse
5. Mettre à jour memory.md si nouvelle décision/contexte

---

## Règles absolues

| # | Règle |
|---|-------|
| 1 | **Jamais charger plus de 7 skills** pour une requête |
| 2 | **Jamais recharger VS Code** — tout se fait dynamiquement |
| 3 | **Toujours reviewer les outputs High/Critical** |
| 4 | **Jamais demander quel skill utiliser** (sauf pertinence Secondaire) |
| 5 | **Purger le contexte inutile** quand on switch de profil |
| 6 | **Tout nouveau skill DOIT passer skill-tester** avant merge |
| 7 | **Broad Search Fallback obligatoire** si pertinence Secondaire |
| 8 | **Cross-Profile Auto** sur les mots-clés ambigus |
| 9 | **Sanity Check** avant chaque réponse — vérifier cohérence contexte |

---

## Profils disponibles

| Profil | Dossier | Experts |
|--------|---------|---------|
| `dev` | `~/.kimi/skills-profiles/dev/` | Stack Principal, Frontend, Backend, Cloud, AI/ML, Perf, Security, Testing, Quality, Planning, Agent Meta |
| `business` | `~/.kimi/skills-profiles/business/` | C-Suite, Finance, Commercial, Operations, Growth, Strategy, Special |
| `marketing` | `~/.kimi/skills-profiles/marketing/` | Content/SEO, Acquisition, Conversion, Launch, Analytics, Channels, Psychology |
| `architecture` | `~/.kimi/skills-profiles/architecture/` | DDD, ADR, RFC, patterns |
| `security` | `~/.kimi/skills-profiles/security/` | Audit, threat model, hardening |
| `design` | `~/.kimi/skills-profiles/design/` | UI/UX, Taste, Figma |

---

## Workflows supportés

| Workflow | Usage | Statut |
|----------|-------|--------|
| **Expert Pool** | Sélection dynamique d'expert (défaut) | ✅ Actif |
| **Broad Search Fallback** | Si pertinence Secondaire | ✅ Actif |
| **Cross-Profile Auto** | Requêtes multi-domaines | ✅ Actif |
| **Producer-Reviewer** | Génération + relecture qualité | ✅ Auto si High/Critical |
| **Cross-Profile Consultation** | Consultation multi-profils | ✅ Actif |
| **Fan-Out** | Brainstorming, études comparatives | ⚠️ Manuel |
| **Pipeline** | Tâches séquentielles complexes | ⚠️ Manuel |
| **Supervisor** | Orchestration multi-experts | ✅ Intégré |
| **Hierarchical Delegation** | Délégation récursive | ❌ Pas pour Kimi |
