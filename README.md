# project-quickstart

Template pour démarrer un projet web moderne avec Claude Code.

**Stack** : React 18 + TypeScript + Vite + Tailwind + shadcn/ui + i18n (4 langues) + Supabase + React Router + TanStack Query + React Hook Form + Zod.

---

## Démarrer

```bash
gh repo clone alex-hotton/project-quickstart mon-projet
cd mon-projet
claude --dangerously-skip-permissions
```

Puis dans Claude Code, **un seul point d'entrée** :

- `/initproject` — setup standard (rapide, projet simple)
- `/initprojectadvanced` — setup standard **+ 7 skills + workflow PRD-first dès l'init**

Pas d'autre commande à apprendre. Claude te guide pour les clés Supabase et la suite.

**Redémarrages de Claude Code attendus** : les MCP et les skills sont chargés au boot du CLI uniquement. Tu vas relancer Claude Code une fois pendant `/initproject` (après le MCP Supabase) et **deux fois** pendant `/initprojectadvanced` (MCP Supabase + skills). À chaque restart, retape la même commande — la détection de Phase 0 reprend automatiquement où tu en étais.

---

## Quelle version choisir ?

Les deux commandes posent le **même setup technique** : stack identique, structure de doc identique (OpenAI Harness), `CLAUDE.md` avec les deux principes d'ingénierie inscrits, repo GitHub privé.

`/initprojectadvanced` ajoute **7 skills + un workflow PRD-first dès le premier jour** :

| Skill | À quoi ça sert |
|---|---|
| `grill-me` | Interroge l'utilisateur jusqu'à compréhension partagée d'une idée |
| `tdd` | Test-driven development discipliné (red-green-refactor vertical) |
| `improve-codebase-architecture` | Refactor en deep modules + maintient `CONTEXT.md` |
| `to-prd` | Synthétise une conversation en PRD GitHub issue |
| `to-issues` | Découpe un PRD en issues GitHub vertical-slice |
| `supabase-postgres-best-practices` | Schémas, indexes, RLS, perf Postgres |
| `frontend-design` | UI distinctive (anti-générique IA) |

**Différence clé du flow** : `/initproject` te demande "colle ton PRD ou décris-moi ton projet" et s'arrête au push GitHub. `/initprojectadvanced` enchaîne :

1. **Phase 7** — te demande comment tu veux faire le PRD :
   - **(A)** `grill-me` ici dans Claude Code (interview interactive) → `to-prd` publie le PRD comme GitHub issue
   - **(B)** Tu colles ton PRD existant (depuis Cowork ou ailleurs) → publié comme GitHub issue
2. **Phase 10** — invoque `to-issues` qui découpe le PRD en GitHub issues vertical-slice
3. **Phase 11** — invoque `tdd` pour livrer la **première feature** en red-green-refactor + ouvre une PR

Tu termines `/initprojectadvanced` avec : un repo GitHub privé, un PRD documenté comme GitHub issue, des slices prêtes à attaquer, **une feature livrée en TDD**.

**Recommandation** : `/initprojectadvanced` par défaut. Tu apprends la méthodo en livrant du code réel, sur ton vrai projet.

---

## Comment les skills sont installés (advanced)

`git clone --depth 1` des 3 repos upstream open-source dans `/tmp`, puis `cp -r` des 7 dossiers de skills dans `.claude/skills/`. Pas de CLI tiers, pas de menu interactif. Toujours la dernière version au moment de l'init. Pour refaire l'install plus tard, relance la phase 5 du command file.

Skills installés (licences open-source préservées dans chaque dossier) :
- 5 skills méthodo : `grill-me`, `tdd`, `improve-codebase-architecture`, `to-prd`, `to-issues` (+ `grill-with-docs` en référence interne)
- 1 skill SQL/RLS : `supabase-postgres-best-practices`
- 1 skill UI : `frontend-design`

---

## Structure générée

```
CLAUDE.md                         # principes [+ méthodologie + skills si advanced] + table des matières
ARCHITECTURE.md                   # vue système
CONTEXT.md                        # vocabulaire de domaine (advanced)
docs/
├── DESIGN.md                     # design system, responsive, SEO
├── FRONTEND.md                   # TS conventions, components, i18n
├── PRODUCT_SENSE.md              # pointer vers le PRD GitHub issue
├── QUALITY_SCORE.md              # lint/typecheck/build/test gates
├── RELIABILITY.md                # error handling, observability
├── SECURITY.md                   # RLS, auth, secrets, migrations
├── PLANS.md                      # index des plans d'exécution
├── design-docs/index.md          # design docs longs
├── exec-plans/{active,completed}/
├── exec-plans/tech-debt-tracker.md
├── adr/README.md                 # architectural decision records
├── product-specs/index.md        # index des PRDs (GitHub issues)
├── generated/README.md           # artefacts auto-générés
└── references/README.md          # llms.txt externes, docs de libs

src/
├── components/{ui,layout,features}/
├── hooks/, lib/, locales/, pages/, services/, types/

supabase/migrations/
```

---

## Ce que contient le `CLAUDE.md` généré

`CLAUDE.md` est lu par Claude au début de chaque session. C'est le **système nerveux** du projet.

**Les deux versions** y inscrivent :

- **Principle 1 — Documentation is the source of truth** : Claude met à jour la doc dans le même commit que le code. Le contenu thématique vit dans `docs/*.md`.
- **Principle 2 — Verify before you ask** : Claude lance `tsc`, `npm run lint`, `npm run build`, ouvre Chrome, etc. avant de demander à l'utilisateur de valider. L'utilisateur n'est sollicité que pour les choix produit, l'esthétique, ou l'ambiguïté métier.
- **Documentation map** : liens vers `ARCHITECTURE.md`, `CONTEXT.md`, et chaque `docs/*.md` thématique.

**`/initprojectadvanced` ajoute en plus** dans `CLAUDE.md` :

- **Methodology** : workflow 5 stages (`grill-me` → `to-prd` → `to-issues` → `tdd` → `improve-codebase-architecture`), quand invoquer chaque skill, anti-hallucinations LLM, "ask the AI for / don't ask the AI for".
- **Skills installed** : table des 7 skills + conventions (issue tracker GitHub, label `needs-triage`).

Pas de fichier `METHODOLOGY.md` séparé — tout est dans `CLAUDE.md` pour que Claude ait l'info à chaque session sans aller la chercher.

---

## Prérequis

- Node 20+
- `git` (pour le clone des skills)
- `gh` CLI authentifié (`gh auth login`)
- Compte Supabase si tu veux un backend (optionnel)

---

## Workflow recommandé (après `/initprojectadvanced`)

```
nouvelle feature → grill-me → to-prd → to-issues → tdd → QA
```

Détaillée dans la section **Methodology** du `CLAUDE.md` du projet généré (advanced uniquement).
