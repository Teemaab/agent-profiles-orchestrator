# Exemple : Architecture d'un SaaS a 10 000 utilisateurs

## La question

> "J'ai un SaaS de facturation. Il faut supporter 10 000 utilisateurs simultanes. Quelle architecture ?"

## Le flux

### 1. Router
Mots-cles detectes : "SaaS", "10 000 utilisateurs", "architecture" → **Profil Architecture** (principal) + **Profil Dev** (secondaire) + **Profil Security** (tertiaire — donnees de paiement implicites).

### 2. Profile Loader
- Charge `memory.md` du profil Architecture (projet : Facturo, stack : a definir)
- Charge `EXPERTS.md` du profil Architecture

### 3. Supervisor
Mots-cles : "10 000 utilisateurs", "architecture" → criticite **High** (5 skills max, Producer-Reviewer recommande).

### 4. Expert Pool
- Expert **System Design** → Tres pertinent (mots-cles : 10 000 utilisateurs, architecture)
- Expert **Cloud Architects** → Pertinent (mots-cles : SaaS, scalable)
- Expert **Domain-Driven Design** → Pertinent (mots-cles : facturation, domaine metier)

### 5. Skills selectionnes
- `aws-solution-architect` (Cloud Architects) — infrastructure cloud
- `tactical-ddd` (Domain-Driven Design) — decoupage des bounded contexts
- `create-rfc` (RFC & ADR) — document de decision architecturale
- `legacy-migration-planner` (Legacy Migration — backup, si migration prevue)
- `security-best-practices` (Security — cross-profile, auth et donnees sensibles)

### 6. Cross-Profile Auto
Le profil Security est consulte en secondaire pour verifier :
- L'architecture inclut-elle l'authentification securisee ?
- Les donnees de paiement sont-elles isolees (PCI scope reduction) ?

Skills secondaires charges :
- `testing-api-security-with-owasp-top-10` (Security)

### 7. Producer-Reviewer
- **Producer** genere l'architecture : diagrammes, technologies, estimations.
- **Reviewer** verifie :
  - Le systeme tient-il 10 000 users (calculs de charge, bottleneck) ?
  - Les bounded contexts sont-ils coherents (facturation, utilisateurs, paiement) ?
  - La securite est-elle integree (pas en option) ?
- Iteration 1 : Reviewer demande de detailler la base de donnees (read replicas, sharding).
- Iteration 2 : Reviewer valide.

### 8. Sanity Check
- Cohérence sujet : l'architecture est bien pour Facturo (SaaS facturation) ?
- Cohérence chiffres : les estimations de cout AWS respectent-elles le budget du projet ?
- Context Null Check : le projet est connu → OK.

### 9. Resultat
Document d'architecture structure :
1. **Contexte** : Facturo, SaaS facturation B2B, 10 000 users simultanes
2. **Bounded Contexts** (DDD) :
   - Facturation (core domain)
   - Utilisateurs (supporting domain)
   - Paiement (generic domain — integration stripe)
   - Notification (generic domain)
3. **Architecture technique** :
   - Frontend : Next.js (SSR pour SEO)
   - API : FastAPI (Python) — stateless, auto-scaling
   - Base de donnees : PostgreSQL (primary) + Redis (cache/session)
   - Read replicas pour les rapports
   - Files : RabbitMQ pour les jobs async (generation PDF, envoi email)
4. **Infrastructure** (AWS) :
   - ECS Fargate pour les conteneurs API
   - RDS PostgreSQL (Multi-AZ) + read replicas
   - CloudFront pour le CDN
   - S3 pour les factures PDF
   - WAF + Shield pour la securite
5. **Securite** :
   - Auth OAuth2 + JWT
   - Donnees de paiement isolees (tokenisation Stripe)
   - OWASP Top 10 couvert
6. **RFC** : document de decision avec trade-offs (serverless vs containers, SQL vs NoSQL)

Pas de melange avec du marketing ou du business. Purement architecture.
