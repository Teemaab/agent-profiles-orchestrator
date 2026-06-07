# Exemple : Lancement d'une startup B2B SaaS

## La question

> "J'ai une idee de SaaS B2B pour automatiser les factures. Comment je le lance ?"

## Le flux

### 1. Router
Mots-cles detectes : "SaaS", "B2B", "lancer" → **Profil Business** (principal) + **Profil Marketing** (secondaire).

### 2. Profile Loader
- Charge `memory.md` du profil Business (vide, pas de projet connu)
- Charge `EXPERTS.md` du profil Business

### 3. Supervisor
Mots-cles : "comment", "lancer" → criticite **Medium** (3 skills max).

### 4. Expert Pool
- Expert **C-Suite & Governance** → Pertinent (mots-cles : SaaS, B2B, lancer)
- Expert **Growth & Sales** → Pertinent (mots-cles : lancer)
- Expert **Strategy & Transformation** → Secondaire

Action : Cross-consultation Business + Marketing.

### 5. Skills selectionnes
- `ceo-advisor` (Business) — validation de l'idee, modelisation
- `launch-strategy` (Marketing) — sequence de lancement
- `pricing-strategist` (Business) — pricing B2B

### 6. Sanity Check
- Context Null Check : la requete ne mentionne aucun projet connu dans `memory.md`.
- **STOP** → "De quel projet parles-tu ? Veux-tu que je cree un profil pour ta startup de facturation B2B ?"

### 7. Resultat
L'agent ne repond pas tout de suite. Il demande clarification. Pas d'hallucination de contexte.

---

## Si l'utilisateur repond

> "Oui, le projet s'appelle Facturo. Cible : PME de 10 a 50 employes. Budget marketing : 5000€."

Le `memory.md` du profil Business est mis a jour :
- Projet : Facturo
- Cible : PME 10-50 employes
- Budget marketing : 5000€

Relance du flux avec le contexte charge :
- `ceo-advisor` valide l'idee (marche des PME, concurrence)
- `pricing-strategist` propose un modele freemium vs. payant
- `launch-strategy` propose une sequence : landing page → beta fermee → Product Hunt

Reponse finalisee et coherente avec le projet Facturo.
