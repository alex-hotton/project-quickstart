# Multi-environnements - Setup `staging` + `main`

Tu es un assistant qui met en place un workflow multi-environnements (staging + production) pour un projet web déjà existant utilisant **React + Supabase + Netlify + GitHub**.

Le but : l'utilisateur ship actuellement direct en prod, ça commence à faire mal, on lui installe un environnement `staging` propre où il pourra tester avant de promouvoir vers `main`.

**Décisions techniques figées (ne pas demander à l'utilisateur) :**

- **2 environnements seulement** : `main` (= la branche par défaut du repo, c'est la prod) + `staging` (intégration).
- **Supabase Branching 2.0** avec une branche persistante `staging`. Pas de 2e projet Supabase. L'utilisateur sera donc en plan **Pro** (~$25/mois) + ~$10/mois pour la branche staging. C'est lui qui paie, c'est assumé.
- **Un seul site Netlify** avec branch deploys. Env vars scopées par contexte (`production` vs `branch:staging`).
- **Workflow git** : `feature/* → PR vers staging → PR vers main`. Direct push interdit sur les deux.
- **3 GitHub Actions** : `deploy-staging.yml` (push staging), `deploy-production.yml` (push main), `lint-migrations.yml` (sur PR).

Suis ce guide étape par étape. **Exécute** les commandes, ne te contente pas de les afficher. Attends la confirmation utilisateur uniquement où c'est explicitement demandé.

---

## ÉTAPE 0 : DÉTECTION DE REPRISE (OBLIGATOIRE)

Avant tout, détermine où l'utilisateur en est. Lance ces vérifications dans l'ordre, en parallèle quand c'est possible :

1. Branche `staging` existe localement ? → `git branch --list staging`
2. Branche `staging` existe sur le remote ? → `git ls-remote --heads origin staging`
3. Fichiers workflows présents ? → `ls .github/workflows/deploy-staging.yml .github/workflows/deploy-production.yml .github/workflows/lint-migrations.yml 2>/dev/null`
4. Secrets GitHub configurés ? → `gh secret list | grep -E "SUPABASE_ACCESS_TOKEN|STAGING_PROJECT_ID|PRODUCTION_PROJECT_ID"`
5. Env vars Netlify staging configurées ? → `netlify env:list --context branch:staging --json 2>/dev/null`
6. CLAUDE.md contient déjà le bloc multi-env ? → `grep -l "Multi-environment rules" CLAUDE.md 2>/dev/null`
7. `docs/MULTI_ENV.md` existe ? → `ls docs/MULTI_ENV.md 2>/dev/null`

**Règles de reprise :**

- **Rien n'existe** → PHASE 1
- **Branche staging existe mais pas de workflows** → PHASE 5
- **Workflows existent mais pas de secrets** → PHASE 4
- **Workflows + secrets OK mais pas d'env Netlify** → PHASE 7
- **Tout existe sauf CLAUDE.md / docs/MULTI_ENV.md** → PHASE 9
- **Tout existe** → propose à l'utilisateur de relancer le smoke test (PHASE 10) ou de quitter

**EXÉCUTE CES VÉRIFICATIONS MAINTENANT** avant de continuer.

---

## PHASE 1 : PRÉ-REQUIS

Le but : refuser de continuer si l'environnement n'est pas sain. Mieux vaut un stop net qu'un setup à moitié cassé.

### Étape 1.1 : Confirmation du dossier

Affiche :

```
On va mettre en place un staging sur ce projet :
  📁 [output de pwd]

C'est bien ce projet-là ? (oui / non + chemin correct)
```

Si l'utilisateur donne un autre path → `cd` dedans et reprends ÉTAPE 0 depuis le nouveau dossier.

### Étape 1.2 : Vérifications environnement

Lance ces checks **en parallèle** et compile un rapport. NE continue PAS si un check échoue — affiche la commande à lancer pour corriger et stoppe.

| Check | Commande | Si KO |
|---|---|---|
| Repo git | `git rev-parse --is-inside-work-tree` | Stop : "Ce dossier n'est pas un repo git." |
| Working tree clean | `git status --porcelain` (doit être vide) | Stop : "Tu as des modifs non commit. Commit ou stash d'abord." |
| Remote GitHub | `gh repo view --json nameWithOwner -q .nameWithOwner` | Stop : "Pas de remote GitHub. Configure-le avec `gh repo create` ou `git remote add origin`." |
| Local main à jour | `git fetch origin && git status -uno` | Stop : "Ta branche par défaut n'est pas à jour. Pull d'abord." |
| `gh` CLI loggé | `gh auth status` | Stop : "Pas loggé sur GitHub. Lance `gh auth login`." |
| `supabase` CLI installée | `supabase --version` | Stop : "Installe la CLI Supabase : `brew install supabase/tap/supabase`." |
| `supabase` CLI loggée | `supabase projects list` (doit pas erreurer) | Stop : "Pas loggé sur Supabase. Lance `supabase login`." |
| `netlify` CLI installée | `netlify --version` | Stop : "Installe la CLI Netlify : `npm install -g netlify-cli`." |
| `netlify` CLI loggée + site lié | `netlify status` | Stop : "Pas de site Netlify lié. Lance `netlify link` dans ce dossier." |
| `package.json` existe | `test -f package.json` | Stop : "Pas de package.json. Cette commande suppose un projet Node." |
| Dépendance Supabase | `grep -q "@supabase/supabase-js" package.json` | Stop : "@supabase/supabase-js n'est pas installé. C'est un pré-requis." |

### Étape 1.3 : Vérification du plan Supabase

L'utilisateur DOIT être en Pro pour que le branching soit dispo. Vérifie :

```bash
# Récupère la liste des projets avec leur plan
supabase projects list --output json
```

Cherche le projet actuellement linké (`supabase status` ou `cat supabase/.temp/project-ref` si dispo). Si tu peux pas détecter automatiquement le plan, pose la question :

```
Pour utiliser Supabase Branching, ton projet Supabase doit être en plan Pro
(ou supérieur). Coût indicatif :
  - Plan Pro : $25/mois (le projet existant)
  - Branche persistante "staging" : ~$10/mois en plus
  - TOTAL : ~$35/mois

Tu confirmes que ton projet est en Pro et que tu acceptes ce coût ? (oui / non)
```

Si non → stop avec un message clair :
```
Pas de souci. Reviens lancer /multienvironnements quand tu seras prêt à passer
en Pro. Ou utilise un workflow plus simple (push direct sur main + tests
rigoureux en local) en attendant.
```

### Étape 1.4 : Détection des variables d'env Supabase

Le projet peut utiliser n'importe quel préfixe (`VITE_*`, `NEXT_PUBLIC_*`, `REACT_APP_*`...). Détecte automatiquement :

```bash
# Cherche les noms de variables Supabase dans .env (et .env.local s'il existe)
grep -E "^[A-Z_]*SUPABASE_URL" .env .env.local 2>/dev/null | head -1
grep -E "^[A-Z_]*SUPABASE_ANON" .env .env.local 2>/dev/null | head -1
```

Stocke en mémoire les noms exacts (ex : `VITE_SUPABASE_URL`, `VITE_SUPABASE_ANON_KEY`). On les réutilisera tels quels côté Netlify pour ne rien casser.

Si rien trouvé :
```
Je ne trouve pas tes variables Supabase dans .env. Donne-moi leurs noms exacts :
  - URL Supabase (ex: VITE_SUPABASE_URL) :
  - Clé anon Supabase (ex: VITE_SUPABASE_ANON_KEY) :
```

### Étape 1.5 : Détection de la branche par défaut

```bash
gh repo view --json defaultBranchRef -q .defaultBranchRef.name
```

Stocke le résultat dans `[DEFAULT_BRANCH]`. C'est la branche prod (le plus souvent `main`, parfois `master`). On ne la renomme pas.

---

## PHASE 2 : RÉCAP + CONFIRMATION

Affiche un récapitulatif clair avant de toucher à quoi que ce soit :

```
═══════════════════════════════════════════════════════════════
  SETUP MULTI-ENVIRONNEMENTS
═══════════════════════════════════════════════════════════════

  Repo         : [OWNER/REPO]
  Prod         : branche [DEFAULT_BRANCH] → site Netlify principal
                                          → projet Supabase principal
  Staging      : branche staging        → branch deploy Netlify
                                        → branche persistante Supabase

  Workflow git :
    feature/xyz → PR → staging → PR → [DEFAULT_BRANCH]
                        ↓                     ↓
                  Deploy staging      Deploy production
                  (auto via GHA)      (auto via GHA)

  Ce que je vais faire :
    1. Activer Supabase Branching (étape manuelle dashboard)
    2. Créer la branche persistante "staging" sur Supabase
    3. Créer la branche git "staging" et la pousser
    4. Configurer 5 secrets GitHub
    5. Activer la protection des branches main + staging
    6. Écrire 3 workflows GitHub Actions
    7. Configurer les variables d'env Netlify par contexte
    8. Activer le branch deploy "staging" sur Netlify
    9. Ajouter les redirect URLs staging dans Supabase Auth
   10. Mettre à jour CLAUDE.md + créer docs/MULTI_ENV.md
   11. Smoke test end-to-end (migration triviale + rollback)

  Coût estimé : ~$10/mois en plus du Pro existant.

═══════════════════════════════════════════════════════════════
```

Demande : **"Tout est OK, je peux commencer ? (oui / non)"**

Si non → stop sans rien toucher.

---

## PHASE 3 : SUPABASE BRANCHING

### Étape 3.1 : Activation manuelle du branching

L'API ne permet pas d'activer le branching, c'est exclusivement par le dashboard. Affiche :

```
🌿 Étape manuelle dans le dashboard Supabase
─────────────────────────────────────────────

1. Va sur https://supabase.com/dashboard/project/<TON_PROJECT_REF>/branches
   (ou : Dashboard → ton projet → Branches dans la sidebar)

2. Clique sur "Enable Branching"

3. Confirme (Supabase peut te demander d'ajouter une carte si pas déjà fait)

Reviens ici et tape "ok" quand c'est activé.
```

Récupère `<TON_PROJECT_REF>` via `supabase status` ou en lisant `supabase/.temp/project-ref` si dispo. Si pas dispo, demande explicitement le project ref de prod.

Attends que l'utilisateur tape "ok".

### Étape 3.2 : Création de la branche persistante staging

Une fois le branching activé :

```bash
supabase --experimental branches create staging --persistent
```

Si la commande demande un project ref → passer `--project-ref <PROD_REF>`.

Récupère le **project ref de la branche staging** :

```bash
supabase --experimental branches list --output json
```

Parse le JSON, trouve l'entrée `name == "staging"`, lis le champ `project_ref`. Stocke dans `[STAGING_PROJECT_REF]`.

### Étape 3.3 : Récupération du DB password de la branche staging

L'API ne donne PAS le password de la branche en clair. Affiche :

```
🔑 Étape manuelle pour récupérer le DB password staging
───────────────────────────────────────────────────────

1. Va sur https://supabase.com/dashboard/project/[STAGING_PROJECT_REF]/settings/database

2. Section "Database password" → clique sur "Reset database password"
   (oui c'est la branche, pas la prod, donc pas de risque)

3. Copie le nouveau password généré

4. Colle-le ici (ne sera ni logué, ni commité, juste utilisé pour configurer
   les secrets GitHub) :
```

Stocke dans `[STAGING_DB_PASSWORD]`.

### Étape 3.4 : DB password de la branche prod

On en a aussi besoin pour le workflow de prod :

```
🔑 DB password de ton projet PROD
─────────────────────────────────

Va sur https://supabase.com/dashboard/project/<PROD_REF>/settings/database

Si tu as déjà ton password noté quelque part (dans 1Password etc.), colle-le.
Sinon "Reset database password" et colle le nouveau.

⚠️  Attention : reset le password va invalider toute connexion existante avec
l'ancien password. Si ton site prod tape directement la DB avec ce password,
tu vas casser des trucs. Préfère retrouver l'ancien si possible.

Password prod :
```

Stocke dans `[PRODUCTION_DB_PASSWORD]`.

Stocke aussi le `[PRODUCTION_PROJECT_REF]` (déjà connu via `supabase status`).

### Étape 3.5 : Récupération de l'access token Supabase

```bash
supabase --version  # pour vérifier que la CLI est bien là
```

Demande à l'utilisateur :

```
🔑 Access token Supabase
─────────────────────────

Va sur https://supabase.com/dashboard/account/tokens

Génère un token nommé "github-actions-[NOM_PROJET]" (full access).

Colle-le ici :
```

Stocke dans `[SUPABASE_ACCESS_TOKEN]`.

### Étape 3.6 : Récupération des credentials staging pour le frontend

Récupère l'URL et la clé anon de la branche staging :

```bash
supabase --experimental branches get staging --output json
```

Parse pour trouver :
- `[STAGING_SUPABASE_URL]` → format `https://[STAGING_PROJECT_REF].supabase.co`
- `[STAGING_SUPABASE_ANON_KEY]` → la clé anon publique

Si la commande ne renvoie pas la clé anon, va la récupérer via :

```bash
supabase projects api-keys --project-ref [STAGING_PROJECT_REF] --output json
```

Cherche l'entrée `name == "anon"` et lis `api_key`.

### Étape 3.7 : Sync du schema prod → staging

Les nouvelles branches Supabase partent **vides**. Il faut leur appliquer le schema prod.

Vérifie ce qu'il y a dans `supabase/migrations/` :

```bash
ls supabase/migrations/ 2>/dev/null | wc -l
```

**Cas A : il y a déjà des migrations.**

```bash
supabase link --project-ref [STAGING_PROJECT_REF] --password "[STAGING_DB_PASSWORD]"
supabase db push
```

Vérifie que la commande termine sans erreur.

**Cas B : pas de migrations (le schema actuel a été fait à la main dans le dashboard prod).**

⚠️ Avant de continuer, dump le schema prod et le sauve comme migration initiale :

```bash
# Link sur prod
supabase link --project-ref [PRODUCTION_PROJECT_REF] --password "[PRODUCTION_DB_PASSWORD]"

# Dump schema-only (pas les données)
mkdir -p supabase/migrations
supabase db dump --schema public --file supabase/migrations/$(date -u +%Y%m%d%H%M%S)_initial_schema.sql

# Re-link sur staging et apply
supabase link --project-ref [STAGING_PROJECT_REF] --password "[STAGING_DB_PASSWORD]"
supabase db push
```

Affiche à l'utilisateur :

```
ℹ️  J'ai dumpé ton schema prod en migration initiale. À partir de maintenant,
   toute modification de schema passe par un nouveau fichier dans
   supabase/migrations/. Pas d'édition directe via le dashboard.
```

### Étape 3.8 : Mise à jour de `supabase/config.toml`

Ajoute le bloc `[remotes.staging]` à la fin du fichier (en créant le fichier si absent) :

```toml
[remotes.staging]
project_id = "[STAGING_PROJECT_REF]"
```

---

## PHASE 4 : SECRETS GITHUB

### Étape 4.1 : Configuration des 5 secrets

Lance ces commandes une par une (jamais en bash combiné — on ne veut pas qu'un secret apparaisse dans un message d'erreur shell) :

```bash
gh secret set SUPABASE_ACCESS_TOKEN --body "[SUPABASE_ACCESS_TOKEN]"
gh secret set STAGING_PROJECT_ID --body "[STAGING_PROJECT_REF]"
gh secret set STAGING_DB_PASSWORD --body "[STAGING_DB_PASSWORD]"
gh secret set PRODUCTION_PROJECT_ID --body "[PRODUCTION_PROJECT_REF]"
gh secret set PRODUCTION_DB_PASSWORD --body "[PRODUCTION_DB_PASSWORD]"
```

### Étape 4.2 : Vérification

```bash
gh secret list
```

Doit lister exactement les 5 secrets. Si l'un manque → relance le `gh secret set` correspondant.

---

## PHASE 5 : BRANCHE STAGING + PROTECTION

### Étape 5.1 : Création de la branche staging

```bash
git checkout [DEFAULT_BRANCH]
git pull origin [DEFAULT_BRANCH]
git checkout -b staging
git push -u origin staging
```

### Étape 5.2 : Branch protection sur main et staging

Pour les deux branches, applique :
- PR obligatoire avant merge
- Status checks requis (le job `lint-migrations` qu'on va créer)
- Pas de force-push
- Pas de suppression de branche

```bash
# Protection sur la branche par défaut (prod)
gh api -X PUT repos/:owner/:repo/branches/[DEFAULT_BRANCH]/protection \
  -F "required_status_checks[strict]=true" \
  -F "required_status_checks[contexts][]=lint-migrations" \
  -F "enforce_admins=false" \
  -F "required_pull_request_reviews[required_approving_review_count]=0" \
  -F "required_pull_request_reviews[dismiss_stale_reviews]=true" \
  -F "restrictions=null" \
  -F "allow_force_pushes=false" \
  -F "allow_deletions=false"
```

```bash
# Protection sur staging (un peu plus light : pas de status check requis car
# c'est l'env d'intégration où on teste justement)
gh api -X PUT repos/:owner/:repo/branches/staging/protection \
  -F "required_status_checks[strict]=true" \
  -F "required_status_checks[contexts][]=lint-migrations" \
  -F "enforce_admins=false" \
  -F "required_pull_request_reviews[required_approving_review_count]=0" \
  -F "required_pull_request_reviews[dismiss_stale_reviews]=true" \
  -F "restrictions=null" \
  -F "allow_force_pushes=false" \
  -F "allow_deletions=false"
```

Note : `required_approving_review_count=0` parce qu'on assume solo dev par défaut. L'utilisateur ajoutera s'il bosse en équipe.

Si l'API renvoie une 404 / 403 → c'est que le repo est public sur un compte gratuit ou que l'utilisateur n'a pas les droits admin. Dans ce cas, log un avertissement et continue (pas bloquant) :

```
⚠️  Je n'ai pas pu activer la branch protection (probablement compte gratuit
   ou repo privé sans droits admin). Pas grave : le reste du setup fonctionne.
   Tu peux activer la protection manuellement plus tard via GitHub UI :
   Settings → Branches → Add branch protection rule.
```

---

## PHASE 6 : GITHUB ACTIONS WORKFLOWS

Crée le dossier si nécessaire :

```bash
mkdir -p .github/workflows
```

### Étape 6.1 : `.github/workflows/lint-migrations.yml`

```yaml
name: lint-migrations

on:
  pull_request:
    paths:
      - 'supabase/migrations/**'
      - 'supabase/config.toml'

jobs:
  lint-migrations:
    runs-on: ubuntu-latest
    timeout-minutes: 10
    steps:
      - uses: actions/checkout@v4

      - uses: supabase/setup-cli@v2
        with:
          version: latest

      - name: Start local Supabase
        run: supabase db start

      - name: Lint migrations
        run: supabase db lint --level error --fail-on error

      - name: Dry-run against staging
        env:
          SUPABASE_ACCESS_TOKEN: ${{ secrets.SUPABASE_ACCESS_TOKEN }}
          SUPABASE_DB_PASSWORD: ${{ secrets.STAGING_DB_PASSWORD }}
        run: |
          supabase link --project-ref ${{ secrets.STAGING_PROJECT_ID }}
          supabase db push --dry-run
```

### Étape 6.2 : `.github/workflows/deploy-staging.yml`

```yaml
name: deploy-staging

on:
  push:
    branches: [staging]
  workflow_dispatch:

concurrency:
  group: deploy-staging
  cancel-in-progress: false

jobs:
  deploy:
    runs-on: ubuntu-latest
    timeout-minutes: 15
    env:
      SUPABASE_ACCESS_TOKEN: ${{ secrets.SUPABASE_ACCESS_TOKEN }}
      SUPABASE_DB_PASSWORD: ${{ secrets.STAGING_DB_PASSWORD }}
      PROJECT_ID: ${{ secrets.STAGING_PROJECT_ID }}
    steps:
      - uses: actions/checkout@v4

      - uses: supabase/setup-cli@v2
        with:
          version: latest

      - name: Link to staging
        run: supabase link --project-ref $PROJECT_ID

      - name: Apply migrations
        run: supabase db push

      - name: Deploy edge functions
        run: |
          if [ -d "supabase/functions" ] && [ "$(ls -A supabase/functions 2>/dev/null)" ]; then
            supabase functions deploy --project-ref $PROJECT_ID
          else
            echo "No edge functions to deploy."
          fi
```

### Étape 6.3 : `.github/workflows/deploy-production.yml`

```yaml
name: deploy-production

on:
  push:
    branches: [[DEFAULT_BRANCH]]
  workflow_dispatch:

concurrency:
  group: deploy-production
  cancel-in-progress: false

jobs:
  deploy:
    runs-on: ubuntu-latest
    timeout-minutes: 15
    environment: production
    env:
      SUPABASE_ACCESS_TOKEN: ${{ secrets.SUPABASE_ACCESS_TOKEN }}
      SUPABASE_DB_PASSWORD: ${{ secrets.PRODUCTION_DB_PASSWORD }}
      PROJECT_ID: ${{ secrets.PRODUCTION_PROJECT_ID }}
    steps:
      - uses: actions/checkout@v4

      - uses: supabase/setup-cli@v2
        with:
          version: latest

      - name: Link to production
        run: supabase link --project-ref $PROJECT_ID

      - name: Apply migrations
        run: supabase db push

      - name: Deploy edge functions
        run: |
          if [ -d "supabase/functions" ] && [ "$(ls -A supabase/functions 2>/dev/null)" ]; then
            supabase functions deploy --project-ref $PROJECT_ID
          else
            echo "No edge functions to deploy."
          fi
```

⚠️ Remplace bien `[DEFAULT_BRANCH]` par la valeur réelle (`main`, `master`, etc.).

### Étape 6.4 : Commit + push des workflows sur staging

On push d'abord sur staging pour ne pas déclencher le workflow production avant qu'on ait validé que tout marche.

```bash
git checkout staging
git add .github/workflows/ supabase/config.toml supabase/migrations/
git commit -m "Add multi-env CI/CD workflows + staging branch config"
git push origin staging
```

Le push sur `staging` va déclencher `deploy-staging.yml`. Surveille :

```bash
gh run watch
```

Si le run échoue → diagnostique l'erreur, corrige, recommit. Tant que le run staging n'est pas vert, ne continue PAS.

---

## PHASE 7 : NETLIFY

### Étape 7.1 : Configuration des env vars par contexte

Les noms de variables sont ceux détectés à l'Étape 1.4 (ex : `VITE_SUPABASE_URL`). Adapte aux noms réels.

```bash
# Production : valeurs déjà existantes (les laisser en place)
# On les liste pour confirmer
netlify env:list --context production
```

Si les valeurs production ne sont pas encore set sur le contexte `production` (elles peuvent être set sur "all contexts") → set-les explicitement :

```bash
netlify env:set [VITE_SUPABASE_URL_VAR_NAME] "<URL_PROD>" --context production
netlify env:set [VITE_SUPABASE_ANON_VAR_NAME] "<ANON_PROD>" --context production
```

Puis ajoute les overrides staging :

```bash
netlify env:set [VITE_SUPABASE_URL_VAR_NAME] "[STAGING_SUPABASE_URL]" --context branch:staging
netlify env:set [VITE_SUPABASE_ANON_VAR_NAME] "[STAGING_SUPABASE_ANON_KEY]" --context branch:staging
```

### Étape 7.2 : Activation du branch deploy `staging`

La CLI Netlify ne permet pas d'activer les branch deploys par branche spécifique → étape manuelle :

```
🌐 Étape manuelle Netlify
─────────────────────────

1. Va sur https://app.netlify.com/sites/<TON_SITE>/configuration/deploys

2. Section "Branches and deploy contexts" → "Configure"

3. Sélectionne "Let me add individual branches"

4. Ajoute "staging" dans la liste

5. Save

Reviens ici et tape "ok" quand c'est fait.
```

Récupère le slug du site via `netlify status`. Attends "ok".

### Étape 7.3 : Vérification

```bash
netlify env:list --context production --json | jq '.[] | select(.key | contains("SUPABASE"))'
netlify env:list --context branch:staging --json | jq '.[] | select(.key | contains("SUPABASE"))'
```

Les valeurs DOIVENT différer entre les deux contextes. Si elles sont identiques → l'override staging n'a pas pris, recommence l'Étape 7.1.

---

## PHASE 8 : SUPABASE AUTH REDIRECT URLS

Cette étape évite que l'OAuth casse en silence sur staging.

### Étape 8.1 : Identifier l'URL staging

Par défaut, le branch deploy Netlify est sur :
```
https://staging--<TON_SITE>.netlify.app
```

Récupère le slug via :
```bash
netlify status --json | jq -r '.siteData.name'
```

Stocke dans `[STAGING_URL]`.

### Étape 8.2 : Ajout dans le dashboard Supabase

```
🔐 Étape manuelle Supabase Auth (staging)
──────────────────────────────────────────

1. Va sur https://supabase.com/dashboard/project/[STAGING_PROJECT_REF]/auth/url-configuration

2. Site URL :
   [STAGING_URL]

3. Redirect URLs : ajoute (en plus de ce qui peut déjà y être)
   [STAGING_URL]/**

4. Save

Tape "ok" quand c'est fait.
```

Note : c'est UNIQUEMENT pour la branche staging Supabase. La prod garde sa config existante (on n'y touche pas).

---

## PHASE 9 : CLAUDE.MD + DOCS

### Étape 9.1 : Bloc à insérer dans `CLAUDE.md`

**Cas A : `CLAUDE.md` existe.**

Lis le fichier. Insère le bloc ci-dessous **juste après le titre H1** (la première ligne `# ...`), avant tout autre contenu.

**Cas B : `CLAUDE.md` n'existe pas.**

Crée-le avec uniquement ce bloc, précédé d'un titre minimal :

```markdown
# [NOM_DU_PROJET — extrait du package.json]

```

Bloc à insérer (en anglais — c'est lu par Claude, c'est cohérent avec les autres conventions) :

```markdown
## Multi-environment rules

This project ships through two environments: `staging` (the integration branch)
and `[DEFAULT_BRANCH]` (production). Both are wired to dedicated Supabase
branches and Netlify deploy contexts. The rules below are non-negotiable.

1. **Never apply migrations directly to production.** Migrations are written
   locally, validated by `supabase db push --dry-run` against staging (the
   `lint-migrations` workflow does this on every PR), pushed to the `staging`
   branch (the `deploy-staging` workflow auto-applies), verified, and only then
   PR'd into `[DEFAULT_BRANCH]`.

2. **Never run `supabase db reset`, `supabase db push`, or any destructive DB
   command against a project linked to production.** Always confirm the linked
   ref with `supabase status` before destructive operations. When in doubt,
   relink to staging: `supabase link --project-ref $STAGING_PROJECT_ID`.

3. **Never hardcode Supabase URLs or anon keys in source code.** All env values
   come from the framework's env mechanism (Vite/Next/etc.). Netlify injects
   different values per deploy context. Hardcoding a URL silently bypasses
   staging.

4. **Never commit service role keys, DB passwords, or `SUPABASE_ACCESS_TOKEN`.**
   These live only in GitHub Actions secrets and Netlify environment variables.
   If you ever paste one in chat, treat it as leaked: rotate it immediately.

5. **The default deploy flow is: `feature/* → PR → staging → PR → [DEFAULT_BRANCH]`.**
   Direct push to `[DEFAULT_BRANCH]` is forbidden by branch protection. If the
   user asks for "a quick fix in prod," push it to `staging` first, verify the
   staging deploy is green, then promote via PR.

6. **New Edge Functions and Storage buckets must be declared in
   `supabase/config.toml`** so both staging and prod stay in sync.

7. **`[DEFAULT_BRANCH]` is production. `staging` is staging. There is no other
   long-lived environment branch unless this rule is explicitly amended here.**

See [docs/MULTI_ENV.md](./docs/MULTI_ENV.md) for the full workflow.
```

Remplace `[DEFAULT_BRANCH]` partout par la valeur réelle.

### Étape 9.2 : Création de `docs/MULTI_ENV.md`

```bash
mkdir -p docs
```

Contenu du fichier :

```markdown
# Multi-environment workflow

This project runs two environments end-to-end:

| Env | Git branch | Supabase | Netlify |
|---|---|---|---|
| Production | `[DEFAULT_BRANCH]` | Main project (`[PRODUCTION_PROJECT_REF]`) | Production deploy |
| Staging | `staging` | Persistent branch (`[STAGING_PROJECT_REF]`) | Branch deploy |

## Daily workflow

```text
feature/foo
   │
   │ commit + push
   ▼
PR ─── lint-migrations ─── merge ──► staging
                                       │
                                       │ deploy-staging.yml runs
                                       │   - supabase db push (staging branch)
                                       │   - supabase functions deploy
                                       │ Netlify branch deploy fires
                                       ▼
                                  Smoke test on staging
                                       │
                                       │ open PR
                                       ▼
PR ─── lint-migrations ─── merge ──► [DEFAULT_BRANCH]
                                       │
                                       │ deploy-production.yml runs
                                       │   - supabase db push (prod project)
                                       │   - supabase functions deploy
                                       │ Netlify production deploy fires
                                       ▼
                                  Live in production
```

## Creating a new feature

```bash
git checkout staging && git pull
git checkout -b feature/my-feature

# ... edit code, write migrations in supabase/migrations/, etc. ...

git add . && git commit -m "feat: my feature"
git push -u origin feature/my-feature
gh pr create --base staging --title "feat: my feature"
```

After staging merge: verify on `https://staging--<site>.netlify.app`, then:

```bash
gh pr create --base [DEFAULT_BRANCH] --head staging --title "Promote staging → production"
```

## Database migrations

All schema changes go through `supabase/migrations/`:

```bash
supabase migration new <description>
# Edit the generated SQL file
git add supabase/migrations/
```

The lint-migrations workflow validates the migration on PR. The deploy-staging
workflow applies it to the staging Supabase branch. The deploy-production
workflow applies it to prod once promoted.

**Never** edit schema via the dashboard. **Never** run `supabase db push`
locally against production.

## Edge functions

Functions in `supabase/functions/<name>/index.ts` are deployed automatically
by both `deploy-staging.yml` and `deploy-production.yml`. No manual deploy.

## Environment variables

| Layer | Where | Per-env? |
|---|---|---|
| Frontend (Supabase URL + anon key) | Netlify env vars, scoped to `production` and `branch:staging` | Yes |
| GitHub Actions secrets | Repo settings → Secrets and variables → Actions | Yes |
| Local dev (`.env`) | Hits production Supabase by default — change to staging URL+key when testing migrations | Manual |

To run locally against staging:

```bash
# Copy staging values from Netlify
netlify env:list --context branch:staging
# Paste the URL/anon key into a .env.staging file
# Run with: VITE_SUPABASE_URL=... VITE_SUPABASE_ANON_KEY=... npm run dev
```

## Manual deploy / rollback

Each workflow has `workflow_dispatch:` enabled, so you can re-run a deploy from
the GitHub Actions tab without pushing a new commit.

To rollback prod, revert the offending commit on `[DEFAULT_BRANCH]` (don't
force-push) — the revert will trigger `deploy-production.yml` again with the
previous state.

## Verifying the setup

```bash
# Both projects exist and CLI is authenticated
supabase projects list
supabase --experimental branches list

# All 5 secrets are configured
gh secret list
# Expected: SUPABASE_ACCESS_TOKEN, STAGING_PROJECT_ID, STAGING_DB_PASSWORD,
#           PRODUCTION_PROJECT_ID, PRODUCTION_DB_PASSWORD

# Branch protection is active
gh api repos/:owner/:repo/branches/[DEFAULT_BRANCH]/protection
gh api repos/:owner/:repo/branches/staging/protection

# Netlify env vars differ between contexts
netlify env:list --context production
netlify env:list --context branch:staging
```
```

Remplace `[DEFAULT_BRANCH]`, `[PRODUCTION_PROJECT_REF]`, `[STAGING_PROJECT_REF]` par les valeurs réelles.

### Étape 9.3 : Commit

```bash
git checkout staging
git add CLAUDE.md docs/MULTI_ENV.md
git commit -m "Add multi-environment rules to CLAUDE.md + docs/MULTI_ENV.md"
git push origin staging
```

---

## PHASE 10 : SMOKE TEST END-TO-END

Le but : prouver que la chaîne complète marche. On crée une migration triviale, on la push, on vérifie qu'elle s'applique sur staging Supabase, on la promeut en prod, on vérifie qu'elle s'applique en prod, puis on rollback.

### Étape 10.1 : Migration triviale sur staging

```bash
git checkout staging && git pull
git checkout -b chore/multienv-smoke-test
```

Crée le fichier `supabase/migrations/$(date -u +%Y%m%d%H%M%S)_smoke_test.sql` :

```sql
CREATE TABLE IF NOT EXISTS public._multienv_smoke_test (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  env text NOT NULL,
  created_at timestamptz DEFAULT now()
);

ALTER TABLE public._multienv_smoke_test ENABLE ROW LEVEL SECURITY;
```

```bash
git add supabase/migrations/
git commit -m "chore: multi-env smoke test migration"
git push -u origin chore/multienv-smoke-test
gh pr create --base staging --title "chore: multi-env smoke test" --body "Validates the multi-env pipeline end-to-end. Will be reverted."
```

### Étape 10.2 : Merge dans staging et vérification

Récupère le numéro de PR via `gh pr view --json number -q .number`. Merge :

```bash
gh pr merge <PR_NUMBER> --squash --delete-branch
```

Watch le workflow staging :

```bash
gh run watch
```

Attends qu'il soit vert. Puis vérifie côté Supabase staging :

```bash
supabase link --project-ref [STAGING_PROJECT_REF] --password "[STAGING_DB_PASSWORD]"
supabase db remote commit --dry-run 2>&1 | head -20
# OU plus simplement :
psql "postgresql://postgres:[STAGING_DB_PASSWORD]@db.[STAGING_PROJECT_REF].supabase.co:5432/postgres" \
  -c "SELECT to_regclass('public._multienv_smoke_test');"
```

Doit retourner `public._multienv_smoke_test` (et pas NULL).

Si NULL → le workflow staging a fail, diagnostique avant de continuer.

Vérifie aussi le branch deploy Netlify :

```bash
netlify api listSiteDeploys --data '{"site_id":"<SITE_ID>"}' \
  | jq '.[] | select(.branch == "staging") | {state, created_at, deploy_url}' \
  | head -20
```

Doit montrer un deploy `state: "ready"` récent sur la branche `staging`.

### Étape 10.3 : Promotion staging → production

```bash
gh pr create --base [DEFAULT_BRANCH] --head staging --title "Promote staging → production (smoke test)" --body "Smoke test verified on staging. Promoting to production."
```

Merge :

```bash
gh pr merge <NEW_PR_NUMBER> --squash
```

Watch le workflow production :

```bash
gh run watch
```

Vérifie côté Supabase prod :

```bash
psql "postgresql://postgres:[PRODUCTION_DB_PASSWORD]@db.[PRODUCTION_PROJECT_REF].supabase.co:5432/postgres" \
  -c "SELECT to_regclass('public._multienv_smoke_test');"
```

Doit retourner `public._multienv_smoke_test`.

### Étape 10.4 : Rollback

On supprime la table proprement (via migration, pas en touchant au dashboard) :

```bash
git checkout [DEFAULT_BRANCH] && git pull
git checkout staging && git pull
git checkout -b chore/multienv-smoke-test-cleanup
```

Crée `supabase/migrations/$(date -u +%Y%m%d%H%M%S)_smoke_test_cleanup.sql` :

```sql
DROP TABLE IF EXISTS public._multienv_smoke_test;
```

```bash
git add supabase/migrations/
git commit -m "chore: cleanup multi-env smoke test"
git push -u origin chore/multienv-smoke-test-cleanup
gh pr create --base staging --title "chore: cleanup smoke test"
gh pr merge <PR_NUMBER> --squash --delete-branch
gh run watch
```

Une fois staging vert, promote :

```bash
gh pr create --base [DEFAULT_BRANCH] --head staging --title "Promote: cleanup smoke test"
gh pr merge <NEW_PR_NUMBER> --squash
gh run watch
```

Vérifie que la table n'existe plus dans les deux envs :

```bash
psql "postgresql://postgres:[STAGING_DB_PASSWORD]@db.[STAGING_PROJECT_REF].supabase.co:5432/postgres" \
  -c "SELECT to_regclass('public._multienv_smoke_test');"
# Doit retourner: NULL

psql "postgresql://postgres:[PRODUCTION_DB_PASSWORD]@db.[PRODUCTION_PROJECT_REF].supabase.co:5432/postgres" \
  -c "SELECT to_regclass('public._multienv_smoke_test');"
# Doit retourner: NULL
```

---

## PHASE 11 : RÉCAPITULATIF FINAL

Affiche :

```
═══════════════════════════════════════════════════════════════
  ✅ MULTI-ENVIRONNEMENTS CONFIGURÉ
═══════════════════════════════════════════════════════════════

  Environnements actifs :
    🟢 production  → branche [DEFAULT_BRANCH]
                   → Supabase project [PRODUCTION_PROJECT_REF]
                   → Netlify production deploy

    🟡 staging     → branche staging
                   → Supabase branch [STAGING_PROJECT_REF]
                   → Netlify branch deploy
                   → URL : [STAGING_URL]

  GitHub Actions :
    ✅ lint-migrations    (sur chaque PR)
    ✅ deploy-staging     (push staging)
    ✅ deploy-production  (push [DEFAULT_BRANCH])

  Branch protection :
    ✅ [DEFAULT_BRANCH]   (PR + status checks requis)
    ✅ staging            (PR + status checks requis)

  Smoke test : ✅ passé end-to-end (staging + prod), nettoyé.

  ───────────────────────────────────────────────────────────

  À partir de maintenant, ton workflow :

    1. Crée une feature branch DEPUIS staging :
         git checkout staging && git pull
         git checkout -b feature/ma-feature

    2. Code, commit, push.

    3. PR vers staging → merge → vérifie sur staging.netlify.app

    4. PR de staging vers [DEFAULT_BRANCH] → merge → live en prod.

  Plus jamais de push direct sur [DEFAULT_BRANCH]. La branch protection
  l'empêche de toute façon.

  ───────────────────────────────────────────────────────────

  Liens utiles :
    📊 GitHub Actions  : https://github.com/[OWNER/REPO]/actions
    🌐 Netlify         : https://app.netlify.com/sites/[SITE]
    🟢 Supabase prod   : https://supabase.com/dashboard/project/[PRODUCTION_PROJECT_REF]
    🟡 Supabase staging: https://supabase.com/dashboard/project/[STAGING_PROJECT_REF]
    📖 Doc complète    : ./docs/MULTI_ENV.md
    📜 Règles AI       : ./CLAUDE.md (section "Multi-environment rules")

═══════════════════════════════════════════════════════════════
```

---

## NOTES IMPORTANTES POUR L'ASSISTANT

- **Tu exécutes les commandes**, tu ne te contentes pas de les afficher. La seule exception : les étapes manuelles dashboard (Supabase Branching enable, DB password reset, Netlify branch deploy enable, Auth redirect URLs) — là tu attends un "ok" de l'utilisateur.
- **Tu ne logs JAMAIS les secrets dans tes messages.** Quand tu reçois un access token / DB password de l'utilisateur, tu confirmes "reçu" et tu l'utilises silencieusement dans les `gh secret set`. Pas de `echo`, pas de `cat`, pas de répétition à l'écran.
- **Si un check fail, tu STOPPES.** Tu ne tentes pas de bypasser. Tu affiches la commande à lancer pour corriger et tu attends.
- **Le smoke test est OBLIGATOIRE.** C'est ce qui prouve que le setup marche vraiment. Pas de raccourci là-dessus.
- **Tu adaptes les noms de variables d'env** à ce que tu as détecté à l'Étape 1.4 (peut être `VITE_*`, `NEXT_PUBLIC_*`, `REACT_APP_*`, etc.). Pas de hardcode.
- **Tu remplaces `[DEFAULT_BRANCH]`** dans les workflows YAML et la doc par la valeur réelle (`main` la plupart du temps).
- **Pour les commits que tu génères**, suis le style du repo (regarde `git log --oneline -10`). Si pas de convention claire, utilise des messages descriptifs simples.
- **Ne touche pas au code applicatif.** Cette commande configure l'infra et la doc, point. Pas de modif dans `src/`.
- **Si l'utilisateur a déjà des migrations** dans `supabase/migrations/`, tu les laisses tranquilles et tu les apply juste sur staging. S'il n'en a pas, tu fais un `db dump` de prod pour créer la migration initiale (Étape 3.7 cas B).
- **À la fin, tu rappelles que le workflow quotidien commence DEPUIS staging**, pas depuis main. C'est la principale erreur que les gens font après ce setup.
