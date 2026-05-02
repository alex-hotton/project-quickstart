# Initialisation de Projet — Version Advanced

Tu es un assistant qui aide à créer un nouveau projet web moderne **avec un workflow d'ingénierie rigoureux**. Suis ce guide étape par étape de manière interactive.

**Différence avec `/initproject`** : exactement le même setup technique (React + TS + Vite + Tailwind + shadcn + i18n + Supabase) **PLUS** :
- 7 skills pré-installés (PRD/refactor/TDD + supabase-postgres + frontend-design)
- Création de `CONTEXT.md` (vocabulaire de domaine, alimenté par le skill `improve-codebase-architecture`)
- Section "Skills installed" dans `CLAUDE.md`
- PRD publié comme GitHub issue dès la Phase 7 (via `grill-me`+`to-prd` ou collé manuellement)

---

## ÉTAPE 0 : DÉTECTION DE REPRISE (OBLIGATOIRE)

**AVANT TOUTE CHOSE**, vérifie l'état du projet pour savoir où reprendre. Le flow comporte **deux redémarrages obligatoires de Claude Code** (après l'ajout du MCP Supabase, et après installation des skills), donc l'utilisateur va relancer `/initprojectadvanced` plusieurs fois — détecte précisément l'étape en cours.

Utilise les outils Glob ou Read pour vérifier :

1. `package.json` existe ? (projet créé)
2. `.env` existe avec les clés Supabase ?
3. `src/services/supabase.ts` existe ? (client Supabase créé)
4. Le MCP Supabase est disponible ? (essaie d'utiliser un outil MCP Supabase, OAuth déjà validé)
5. Les skills sont-ils installés ? Vérifie en exécutant :
   ```bash
   ls .claude/skills/grill-me/SKILL.md 2>/dev/null && \
   ls .claude/skills/tdd/SKILL.md 2>/dev/null && \
   ls .claude/skills/to-prd/SKILL.md 2>/dev/null && \
   ls .claude/skills/to-issues/SKILL.md 2>/dev/null && \
   ls .claude/skills/improve-codebase-architecture/SKILL.md 2>/dev/null && \
   ls .claude/skills/supabase-postgres-best-practices/SKILL.md 2>/dev/null && \
   ls .claude/skills/frontend-design/SKILL.md 2>/dev/null
   ```
   Si tout existe, les skills sont installés. Sinon, certains manquent ou sont dans `~/.claude/skills/` (refais le check à cet endroit).
6. `CLAUDE.md` existe ? (doc générée)

**Règles de reprise (en ordre, prends la première qui matche) :**

| État détecté | Reprends à |
|---|---|
| Une PR existe déjà sur le repo (`gh pr list` non vide) | **PROJET INIT + 1ère FEATURE LIVRÉE** — dis-le et stop |
| `CLAUDE.md` existe + plus d'1 issue `needs-triage` (PRD + slices créés) mais pas de PR | **PHASE 11** (tdd sur la 1ère issue) |
| `CLAUDE.md` existe + exactement 1 issue `needs-triage` (PRD seul, pas de slices) | **PHASE 10** (to-issues) |
| GitHub repo existe + une issue `needs-triage` mais pas de `CLAUDE.md` | **PHASE 8** (génération doc) |
| GitHub repo existe (`gh repo view` OK) mais pas d'issue PRD | **PHASE 7** (grill-me + to-prd) |
| Skills installés (check 5 = OK) mais pas de GitHub repo | **PHASE 6** (création repo GitHub) |
| `.env` existe + MCP Supabase disponible + test passé mais skills pas installés | **PHASE 5** (install skills) |
| `.env` existe + MCP Supabase disponible mais pas encore testé | **PHASE 4** (test validation Supabase) |
| `package.json` existe mais pas `.env` (ou MCP Supabase indisponible) | **PHASE 3** (config Supabase) |
| Rien n'existe | **PHASE 1** |

Commandes de check :
- Repo GitHub : `gh repo view --json url,visibility 2>/dev/null` (failed → pas de repo)
- Issues : `gh issue list --label needs-triage --json number,title 2>/dev/null` (compte les lignes)
- PR : `gh pr list --json number 2>/dev/null` (vide → pas de PR)

**EXÉCUTE CES VÉRIFICATIONS MAINTENANT** avant de continuer. Annonce à l'user où tu reprends :
```
État détecté : [résumé]
Reprise à la [PHASE X].
```

---

## PHASE 1 : COLLECTE D'INFORMATIONS

### Étape 1.1 : Détection du nom du projet

Le nom du projet est le nom du dossier courant.

Utilise `pwd` pour extraire le nom du dossier. Confirme à l'utilisateur :
```
Nom du projet détecté : [NOM_DU_DOSSIER]
```

### Étape 1.2 : Questions sur le backend

Pose ces questions à l'utilisateur en utilisant le tool AskUserQuestion :

1. **Backend Supabase** : "Tu as besoin d'un backend (base de données, auth, storage) ?"
   - Oui, j'ai besoin d'un backend
   - Non, juste du frontend

2. **Si backend = Oui**, demande :
   - "Tu veux l'authentification utilisateurs ?" (oui/non)
   - "Tu veux le storage de fichiers ?" (oui/non)

---

## PHASE 2 : SETUP TECHNIQUE

### Étape 2.1 : Nettoyage et préparation

```bash
rm -rf templates/
rm -f README.md
```

### Étape 2.2 : Création du projet (manuel)

Crée `package.json` :

```json
{
  "name": "[NOM_PROJET]",
  "private": true,
  "version": "0.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "tsc -b && vite build",
    "lint": "eslint .",
    "preview": "vite preview"
  }
}
```

Crée `index.html` :

```html
<!doctype html>
<html lang="fr">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" type="image/svg+xml" href="/vite.svg" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>[NOM_PROJET]</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```

Crée `src/main.tsx` :

```tsx
import { StrictMode } from 'react'
import { createRoot } from 'react-dom/client'
import './index.css'
import App from './App.tsx'

createRoot(document.getElementById('root')!).render(
  <StrictMode>
    <App />
  </StrictMode>,
)
```

Crée `src/App.tsx` (temporaire) :

```tsx
function App() {
  return (
    <div>
      <h1>Setup en cours...</h1>
    </div>
  )
}

export default App
```

Crée `src/vite-env.d.ts` :

```ts
/// <reference types="vite/client" />
```

Crée `tsconfig.json` :

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

Crée `tsconfig.node.json` :

```json
{
  "compilerOptions": {
    "composite": true,
    "skipLibCheck": true,
    "module": "ESNext",
    "moduleResolution": "bundler",
    "allowSyntheticDefaultImports": true,
    "strict": true
  },
  "include": ["vite.config.ts"]
}
```

Crée `vite.config.ts` :

```ts
import path from "path"
import react from "@vitejs/plugin-react"
import { defineConfig } from "vite"

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "./src"),
    },
  },
})
```

### Étape 2.3 : Installation des dépendances

```bash
npm install react react-dom
npm install -D vite @vitejs/plugin-react typescript @types/react @types/react-dom
npm install react-router-dom
npm install @tanstack/react-query
npm install react-hook-form zod @hookform/resolvers
npm install react-i18next i18next i18next-browser-languagedetector
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
npm install tailwindcss-animate class-variance-authority clsx tailwind-merge
npm install lucide-react
npm install @radix-ui/react-slot
npm install -D @types/node
```

Si backend Supabase :
```bash
npm install @supabase/supabase-js
```

### Étape 2.4 : Configuration Tailwind

Remplace `tailwind.config.js` :

```js
/** @type {import('tailwindcss').Config} */
export default {
  darkMode: ["class"],
  content: [
    "./index.html",
    "./src/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    container: {
      center: true,
      padding: "2rem",
      screens: { "2xl": "1400px" },
    },
    extend: {
      colors: {
        border: "hsl(var(--border))",
        input: "hsl(var(--input))",
        ring: "hsl(var(--ring))",
        background: "hsl(var(--background))",
        foreground: "hsl(var(--foreground))",
        primary: {
          DEFAULT: "hsl(var(--primary))",
          foreground: "hsl(var(--primary-foreground))",
        },
        secondary: {
          DEFAULT: "hsl(var(--secondary))",
          foreground: "hsl(var(--secondary-foreground))",
        },
        destructive: {
          DEFAULT: "hsl(var(--destructive))",
          foreground: "hsl(var(--destructive-foreground))",
        },
        muted: {
          DEFAULT: "hsl(var(--muted))",
          foreground: "hsl(var(--muted-foreground))",
        },
        accent: {
          DEFAULT: "hsl(var(--accent))",
          foreground: "hsl(var(--accent-foreground))",
        },
        popover: {
          DEFAULT: "hsl(var(--popover))",
          foreground: "hsl(var(--popover-foreground))",
        },
        card: {
          DEFAULT: "hsl(var(--card))",
          foreground: "hsl(var(--card-foreground))",
        },
      },
      borderRadius: {
        lg: "var(--radius)",
        md: "calc(var(--radius) - 2px)",
        sm: "calc(var(--radius) - 4px)",
      },
    },
  },
  plugins: [require("tailwindcss-animate")],
}
```

### Étape 2.5 : Configuration CSS

Remplace `src/index.css` :

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    --background: 0 0% 100%;
    --foreground: 222.2 84% 4.9%;
    --card: 0 0% 100%;
    --card-foreground: 222.2 84% 4.9%;
    --popover: 0 0% 100%;
    --popover-foreground: 222.2 84% 4.9%;
    --primary: 222.2 47.4% 11.2%;
    --primary-foreground: 210 40% 98%;
    --secondary: 210 40% 96.1%;
    --secondary-foreground: 222.2 47.4% 11.2%;
    --muted: 210 40% 96.1%;
    --muted-foreground: 215.4 16.3% 46.9%;
    --accent: 210 40% 96.1%;
    --accent-foreground: 222.2 47.4% 11.2%;
    --destructive: 0 84.2% 60.2%;
    --destructive-foreground: 210 40% 98%;
    --border: 214.3 31.8% 91.4%;
    --input: 214.3 31.8% 91.4%;
    --ring: 222.2 84% 4.9%;
    --radius: 0.5rem;
  }

  .dark {
    --background: 222.2 84% 4.9%;
    --foreground: 210 40% 98%;
    --card: 222.2 84% 4.9%;
    --card-foreground: 210 40% 98%;
    --popover: 222.2 84% 4.9%;
    --popover-foreground: 210 40% 98%;
    --primary: 210 40% 98%;
    --primary-foreground: 222.2 47.4% 11.2%;
    --secondary: 217.2 32.6% 17.5%;
    --secondary-foreground: 210 40% 98%;
    --muted: 217.2 32.6% 17.5%;
    --muted-foreground: 215 20.2% 65.1%;
    --accent: 217.2 32.6% 17.5%;
    --accent-foreground: 210 40% 98%;
    --destructive: 0 62.8% 30.6%;
    --destructive-foreground: 210 40% 98%;
    --border: 217.2 32.6% 17.5%;
    --input: 217.2 32.6% 17.5%;
    --ring: 212.7 26.8% 83.9%;
  }
}

@layer base {
  * {
    @apply border-border;
  }
  body {
    @apply bg-background text-foreground;
  }
}
```

### Étape 2.6 : Structure des dossiers

```
src/
├── components/
│   ├── ui/
│   ├── layout/
│   └── features/
├── hooks/
├── lib/
│   └── utils.ts
├── locales/
│   ├── fr/translation.json
│   ├── en/translation.json
│   ├── de/translation.json
│   └── lu/translation.json
├── pages/
├── services/
├── types/
└── App.tsx

supabase/
└── migrations/
```

Crée `src/lib/utils.ts` :

```ts
import { type ClassValue, clsx } from "clsx"
import { twMerge } from "tailwind-merge"

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs))
}
```

### Étape 2.7 : Configuration i18n

Crée `src/lib/i18n.ts` :

```ts
import i18n from 'i18next'
import { initReactI18next } from 'react-i18next'
import LanguageDetector from 'i18next-browser-languagedetector'

import fr from '@/locales/fr/translation.json'
import en from '@/locales/en/translation.json'
import de from '@/locales/de/translation.json'
import lu from '@/locales/lu/translation.json'

i18n
  .use(LanguageDetector)
  .use(initReactI18next)
  .init({
    resources: {
      fr: { translation: fr },
      en: { translation: en },
      de: { translation: de },
      lu: { translation: lu },
    },
    fallbackLng: 'fr',
    interpolation: { escapeValue: false },
  })

export default i18n
```

Fichiers de traduction :

`src/locales/fr/translation.json` :
```json
{ "common": { "loading": "Chargement...", "error": "Une erreur est survenue", "save": "Enregistrer", "cancel": "Annuler", "delete": "Supprimer", "edit": "Modifier", "back": "Retour" } }
```

`src/locales/en/translation.json` :
```json
{ "common": { "loading": "Loading...", "error": "An error occurred", "save": "Save", "cancel": "Cancel", "delete": "Delete", "edit": "Edit", "back": "Back" } }
```

`src/locales/de/translation.json` :
```json
{ "common": { "loading": "Laden...", "error": "Ein Fehler ist aufgetreten", "save": "Speichern", "cancel": "Abbrechen", "delete": "Löschen", "edit": "Bearbeiten", "back": "Zurück" } }
```

`src/locales/lu/translation.json` :
```json
{ "common": { "loading": "Lueden...", "error": "E Feeler ass opgetrueden", "save": "Späicheren", "cancel": "Ofbriechen", "delete": "Läschen", "edit": "Änneren", "back": "Zréck" } }
```

### Étape 2.8 : Création du .gitignore

```gitignore
# Dependencies
node_modules/

# Build
dist/
build/

# Environment
.env
.env.local
.env.*.local

# IDE
.idea/
.vscode/
*.swp
*.swo

# OS
.DS_Store
Thumbs.db

# Logs
*.log
npm-debug.log*

# Testing
coverage/

# Misc
*.local
```

---

## PHASE 3 : CONFIGURATION SUPABASE (si demandé)

### Étape 3.1 : Création du projet Supabase

Demande à l'utilisateur : "Est-ce que tu as déjà créé ton projet Supabase ?"

- Si **non** :
  ```
  Va sur https://supabase.com et :
  1. Crée un compte (ou connecte-toi)
  2. Clique sur "New Project"
  3. Choisis un nom et un mot de passe pour la DB
  4. Attends que le projet soit prêt (~2 min)

  Reviens me voir quand c'est fait !
  ```

- Si **oui** : Continue

### Étape 3.2 : Récupération des clés

Dis à l'utilisateur :
```
J'ai besoin de 2 informations de ton projet Supabase :

1. URL du projet
   → Settings > Data API
   → Copie l'URL (format: https://xxx.supabase.co)

2. Clé anon/public
   → API Keys > Legacy
   → Copie la clé "anon" (commence par eyJ...)

Colle-moi ces 2 valeurs !
```

### Étape 3.3 : Création du .env

```env
VITE_SUPABASE_URL=https://xxx.supabase.co
VITE_SUPABASE_ANON_KEY=eyJ...
```

### Étape 3.4 : Client Supabase

Crée `src/services/supabase.ts` :

```ts
import { createClient } from '@supabase/supabase-js'

const supabaseUrl = import.meta.env.VITE_SUPABASE_URL
const supabaseAnonKey = import.meta.env.VITE_SUPABASE_ANON_KEY

if (!supabaseUrl || !supabaseAnonKey) {
  throw new Error('Missing Supabase environment variables')
}

export const supabase = createClient(supabaseUrl, supabaseAnonKey)
```

### Étape 3.5 : Hook d'authentification (si demandé)

Crée `src/hooks/useAuth.ts` :

```ts
import { useEffect, useState } from 'react'
import { User } from '@supabase/supabase-js'
import { supabase } from '@/services/supabase'

export function useAuth() {
  const [user, setUser] = useState<User | null>(null)
  const [loading, setLoading] = useState(true)

  useEffect(() => {
    supabase.auth.getSession().then(({ data: { session } }) => {
      setUser(session?.user ?? null)
      setLoading(false)
    })

    const { data: { subscription } } = supabase.auth.onAuthStateChange((_event, session) => {
      setUser(session?.user ?? null)
    })

    return () => subscription.unsubscribe()
  }, [])

  const signIn = async (email: string, password: string) => {
    const { error } = await supabase.auth.signInWithPassword({ email, password })
    if (error) throw error
  }

  const signUp = async (email: string, password: string) => {
    const { error } = await supabase.auth.signUp({ email, password })
    if (error) throw error
  }

  const signOut = async () => {
    const { error } = await supabase.auth.signOut()
    if (error) throw error
  }

  return { user, loading, signIn, signUp, signOut }
}
```

### Étape 3.6 : Connexion du MCP Supabase (OAuth)

Dis à l'utilisateur :
```
Maintenant on connecte le MCP Supabase à Claude Code via OAuth (pas de token à manipuler).

Fais ceci :
1. Quitte Claude Code avec : /exit
2. Dans ton terminal, exécute cette commande :

   claude mcp add --scope project --transport http supabase "https://mcp.supabase.com/mcp"

3. Relance Claude Code avec : claude --dangerously-skip-permissions
4. Tape : /mcp
5. Valide l'authentification Supabase dans le navigateur quand on te le demande
   (une fenêtre va s'ouvrir automatiquement pour te connecter à Supabase)
6. Tape /initprojectadvanced pour reprendre

Je détecterai automatiquement où tu en es et je continuerai.
```

**STOP ICI** - Attends que l'utilisateur relance Claude Code.

---

## PHASE 4 : TEST DE VALIDATION SUPABASE

### Étape 4.1 : Vérification du MCP Supabase

Dis à l'utilisateur : "Je vais vérifier que le MCP Supabase est bien configuré."

Essaie d'utiliser le MCP Supabase. Si ça ne marche pas, dis :
```
Le MCP Supabase n'est pas accessible. Vérifie :

1. As-tu bien exécuté la commande d'ajout du MCP ?
   → claude mcp add --scope project --transport http supabase "https://mcp.supabase.com/mcp"
2. As-tu bien relancé Claude Code après ?
3. As-tu validé l'authentification Supabase dans le navigateur via /mcp ?

Si ce n'est pas encore fait :
→ /exit
→ claude mcp add --scope project --transport http supabase "https://mcp.supabase.com/mcp"
→ claude --dangerously-skip-permissions
→ /mcp et valide l'authentification Supabase dans le navigateur
→ /initprojectadvanced
```

### Étape 4.2 : Création de la table de test

```sql
CREATE TABLE _quickstart_test (
  id SERIAL PRIMARY KEY,
  message TEXT NOT NULL,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

INSERT INTO _quickstart_test (message) VALUES
  ('Setup réussi !'),
  ('Connexion Supabase OK'),
  ('Tu peux commencer ton projet');

ALTER TABLE _quickstart_test ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Allow public read" ON _quickstart_test FOR SELECT USING (true);
```

### Étape 4.3 : Application de test

Remplace `src/App.tsx` (même contenu que `/initproject`, app de test avec tableau Supabase). Voir `.claude/commands/initproject.md` étape 4.3 pour le code complet, copie-le ici.

### Étape 4.4 : Lancement et confirmation

```bash
npm run dev
```

Demande à l'utilisateur : "Est-ce que tu vois le tableau de test avec les 3 messages ?"

Si oui → continue. Si non → diagnostique avec lui.

### Étape 4.5 : Suppression de la table de test

```sql
DROP TABLE IF EXISTS _quickstart_test;
```

---

## PHASE 5 : INSTALLATION DES SKILLS

**C'est ici que la version advanced diverge de `/initproject`.** Méthodo : on `git clone` les 3 repos upstream dans `/tmp`, on copie les dossiers de skills voulus dans `.claude/skills/`, on nettoie. Pas de CLI tiers, pas de menu interactif, pas de réseau au-delà de `git clone`. Toujours la dernière version au moment de l'init.

### Étape 5.1 : Annonce à l'utilisateur

Dis :
```
Tout est connecté côté Supabase. ✅

Maintenant on installe les 7 skills qui vont rendre ton dev plus rigoureux :

5 skills méthodo (workflow PRD → tests → refactor) :
- grill-me                         : interview avant d'implémenter
- tdd                              : test-driven development discipliné
- improve-codebase-architecture    : refactor + maintien de CONTEXT.md
- to-prd                           : génère un PRD GitHub issue
- to-issues                        : casse un PRD en issues vertical-slice

2 skills bonus :
- supabase-postgres-best-practices : RLS, indexes, perf
- frontend-design                  : UI distinctive, anti-générique IA

Je clone les 3 repos upstream et je copie les dossiers voulus.
Aucune intervention de ta part.
```

### Étape 5.2 : Installation via git clone + cp

Exécute ce bloc en une seule séquence Bash. Si une étape échoue (réseau coupé, repo déplacé), arrête et alerte l'utilisateur — ne continue pas avec une install partielle.

```bash
set -e
mkdir -p .claude/skills

# Clone temporaire des 3 repos upstream
TMP=$(mktemp -d)
trap "rm -rf $TMP" EXIT

git clone --depth 1 https://github.com/mattpocock/skills "$TMP/mp" 2>&1
git clone --depth 1 https://github.com/anthropics/skills "$TMP/an" 2>&1
git clone --depth 1 https://github.com/supabase/agent-skills "$TMP/sb" 2>&1

# 5 skills méthodo
cp -r "$TMP/mp/skills/productivity/grill-me" .claude/skills/
cp -r "$TMP/mp/skills/engineering/tdd" .claude/skills/
cp -r "$TMP/mp/skills/engineering/improve-codebase-architecture" .claude/skills/
cp -r "$TMP/mp/skills/engineering/to-prd" .claude/skills/
cp -r "$TMP/mp/skills/engineering/to-issues" .claude/skills/

# improve-codebase-architecture référence ../grill-with-docs/CONTEXT-FORMAT.md et ADR-FORMAT.md
# On bundle grill-with-docs aussi pour que les liens internes ne cassent pas
cp -r "$TMP/mp/skills/engineering/grill-with-docs" .claude/skills/

# Skill anthropics (1)
cp -r "$TMP/an/skills/frontend-design" .claude/skills/

# Skill supabase (1)
cp -r "$TMP/sb/skills/supabase-postgres-best-practices" .claude/skills/

# Vérification
echo "Skills installés :"
ls .claude/skills/
```

### Étape 5.3 : Vérification post-install

Vérifie que les 8 dossiers attendus existent (7 skills + grill-with-docs en référence) :

```bash
for skill in grill-me tdd improve-codebase-architecture to-prd to-issues grill-with-docs frontend-design supabase-postgres-best-practices; do
  if [ -f ".claude/skills/$skill/SKILL.md" ]; then
    echo "✓ $skill"
  else
    echo "✗ $skill MANQUANT"
  fi
done
```

Si quelque chose manque, **arrête-toi et diagnostique** avant de continuer. Causes probables : repo upstream déplacé/renommé, problème réseau, permissions disque.

### Étape 5.4 : Redémarrage de Claude Code

**IMPORTANT** : Les skills sont sur le disque mais Claude Code ne les charge qu'au démarrage. Comme pour le MCP Supabase à la Phase 3, il faut relancer.

Dis à l'utilisateur (texte exact) :

```
Les 7 skills sont sur le disque. ✅

Claude Code les charge au démarrage seulement. Relance maintenant :

  1. /exit
  2. claude --dangerously-skip-permissions
  3. /skills           ← tu dois voir les 7 skills listés
  4. /initprojectadvanced

  Skills attendus dans /skills :
    - grill-me
    - tdd
    - improve-codebase-architecture
    - to-prd
    - to-issues
    - supabase-postgres-best-practices
    - frontend-design

  (grill-with-docs apparaîtra aussi : c'est une dépendance interne
   de improve-codebase-architecture, ne le supprime pas.)

Quand tu relances /initprojectadvanced, je détecte que les skills
sont installés et je passe direct à la Phase 6 (PRD + génération doc).
```

**STOP ICI** — Attends que l'utilisateur relance Claude Code et invoque `/initprojectadvanced` à nouveau. La détection de reprise (Phase 0) renverra à la Phase 6.

### Étape 5.5 : Skip du setup `/setup-matt-pocock-skills`

L'upstream distribue un skill `setup-matt-pocock-skills` qui demande interactivement quel issue tracker utiliser et quels labels de triage. **On le skip volontairement** : on inscrit les conventions directement dans `CLAUDE.md` à la Phase 7 (issue tracker = GitHub, label = `needs-triage`, doc location = `docs/`). Les skills `to-prd` et `to-issues` lisent ces conventions depuis `CLAUDE.md`. Si l'utilisateur veut customiser plus tard, il peut le lancer à la main.

---

## PHASE 6 : CRÉATION DU REPO GITHUB (vide)

On crée le repo GitHub MAINTENANT, avant le PRD. Pourquoi : le skill `to-prd` publie le PRD comme **GitHub issue**, donc le repo doit exister avant qu'on lance to-prd.

### Étape 6.1 : Création du repo

```bash
# Si un remote 'origin' existe (héritage du template), retire-le
git remote remove origin 2>/dev/null || true

# Crée le repo GitHub privé, sans push (le code partira au commit final)
gh repo create [NOM_PROJET] --private --source=. --remote=origin
```

Le repo existe sur GitHub mais est vide. Le code sera pushé à la Phase 9.

### Étape 6.2 : Vérification

```bash
gh repo view --json url,visibility -q '.url + " (" + .visibility + ")"'
```

Tu dois voir l'URL du repo et `PRIVATE`. Si erreur, vérifie l'auth `gh` (`gh auth status`).

---

## PHASE 7 : PRD COMME GITHUB ISSUE

**Objectif** : produire un PRD publié comme GitHub issue (avec label `needs-triage`) sur le repo de Phase 6. Le PRD étant une issue, `to-issues` (Phase 10) pourra l'attaquer ensuite.

Deux chemins possibles selon où l'utilisateur veut faire le PRD. **Demande-lui** au début de la phase.

### Étape 7.1 : Choix de la méthode (AskUserQuestion)

Utilise le tool `AskUserQuestion` pour poser :

> **Question** : "Tu veux faire ton PRD comment ?"
>
> - **Option A** : "Avec `grill-me` ici, dans Claude Code (interview interactive)"
> - **Option B** : "J'ai déjà un PRD (dans Cowork ou ailleurs), je vais le coller"

Branche selon la réponse.

---

### Chemin A — `grill-me` + `to-prd`

#### Étape 7.A.1 : Invocation du skill `grill-me`

Dis :
```
On commence l'interview. Décris-moi ton projet en quelques phrases (ou
tape "grille-moi" et je démarre les questions à froid).
```

Une fois que l'utilisateur a donné un point de départ, **invoque le skill `grill-me`** et conduis l'interview en suivant exactement ses instructions :
- Une question à la fois
- Pour chaque question, propose une réponse recommandée
- Si une question peut être résolue en explorant le code, fais-le au lieu de demander
- Continue jusqu'à compréhension partagée complète

À ce stade le code est un scaffold React + Supabase — explore-le pour ancrer les questions, mais l'essentiel portera sur la **logique métier**.

#### Étape 7.A.2 : Invocation du skill `to-prd`

Quand l'interview converge, **invoque le skill `to-prd`**. Il va :
- Synthétiser la conversation en PRD selon son template (Problem, Solution, User Stories, Implementation Decisions, Testing Decisions, Out of Scope, Further Notes)
- **Publier le PRD comme GitHub issue** avec le label `needs-triage`

Le skill demandera confirmation avant publication. Confirme.

→ Saute à l'étape 7.3.

---

### Chemin B — PRD collé par l'utilisateur

#### Étape 7.B.1 : Demande du PRD

Dis :
```
Colle ton PRD ici (depuis Cowork ou n'importe quel autre outil).
- Markdown OK
- N'importe quelle longueur OK
- Quand tu as fini, envoie ton message
```

Attends que l'utilisateur colle son contenu.

#### Étape 7.B.2 : Extraction du titre

Demande à l'utilisateur (ou propose) un titre court pour l'issue (max ~80 chars). Exemple : "PRD: [nom du projet ou de la feature principale]". Si évident depuis le PRD, propose-le directement et attends confirmation rapide.

#### Étape 7.B.3 : Publication comme GitHub issue

Crée le label `needs-triage` s'il n'existe pas, puis publie le PRD comme issue :

```bash
gh label create needs-triage --color FBCA04 --description "Awaiting triage" 2>/dev/null || true

gh issue create \
  --title "[TITRE_DU_PRD]" \
  --body "$(cat <<'PRD_EOF'
[CONTENU_COLLÉ_PAR_L_UTILISATEUR]
PRD_EOF
)" \
  --label needs-triage
```

→ Continue à l'étape 7.3.

---

### Étape 7.3 : Référence le PRD pour la suite (commun aux deux chemins)

Récupère l'URL, le numéro et le titre de l'issue créée :

```bash
gh issue list --label needs-triage --limit 1 --json number,url,title
```

Note ces valeurs — elles seront référencées dans `docs/PRODUCT_SENSE.md` et `docs/product-specs/index.md` à la Phase 8, et utilisées par `to-issues` à la Phase 10.

Confirme à l'utilisateur :
```
PRD publié : [URL_ISSUE]

Maintenant je génère la doc du projet en intégrant le contexte du PRD.
```

---

## PHASE 8 : GÉNÉRATION DE LA DOCUMENTATION

Structure **OpenAI Harness Engineering** : `CLAUDE.md` à la racine est une table des matières + principes, le contenu vit dans `docs/*.md`. Plus `CONTEXT.md` à la racine pour le vocabulaire de domaine (alimenté par `improve-codebase-architecture`).

À ce stade tu as :
- Le repo GitHub (Phase 6)
- Le PRD publié comme issue (Phase 7) — son URL et son contenu sont dans le contexte de la conversation

### Étape 8.1 : Création de l'arborescence docs

```bash
mkdir -p docs/design-docs docs/exec-plans/active docs/exec-plans/completed docs/generated docs/product-specs docs/references docs/adr
```

### Étape 8.2 : Création de `CLAUDE.md`

`CLAUDE.md` est le **système nerveux du projet**. Lu par Claude au début de chaque session. Il contient TOUT ce dont Claude a besoin pour bosser correctement : principes d'ingénierie + méthodologie + carte de la doc + skills disponibles. Pas de fichier METHODOLOGY.md séparé — la méthodo vit ici.

```markdown
# [NOM_PROJET]

[DESCRIPTION_COURTE_DU_PROJET]

---

## Principles

### 1. Documentation is the source of truth

Update docs in the same commit as the code. A change without a doc update is incomplete.

- `CLAUDE.md` is the source of truth — principles, methodology, TOC, skills. Read it at the start of every session.
- `ARCHITECTURE.md` is the system at a glance.
- `CONTEXT.md` is the domain vocabulary. Maintained by the `improve-codebase-architecture` skill — extend it when new concepts emerge.
- One topic per file in `docs/`. Lead with the why, then the how.
- Short sections, concrete examples, no padding.
- Decisions go in `docs/adr/`. Active plans in `docs/exec-plans/active/`, archived to `completed/` when done.
- Generated artifacts in `docs/generated/` are regenerated, never hand-edited.
- Product specs are GitHub issues; `docs/product-specs/index.md` indexes them.
- Add a doc → link it from this file. Touch a feature → update its doc.

### 2. Verify before you ask

Use your tools before asking the user to test, review, or confirm. The user is for judgment calls, not for what you can check yourself.

| You changed... | Run...                                           |
|----------------|--------------------------------------------------|
| TS code        | `tsc --noEmit`, `npm run lint`                   |
| The build      | `npm run build`                                  |
| UI             | Open in Chrome, exercise the flow                |
| Logic          | Write/run a test                                 |
| A bug fix      | Reproduce, fix, reproduce again — fixed?         |
| DB schema      | Inspect via Supabase MCP                         |
| An assumption  | `grep` / read before you claim it                |

Ask the user only for product decisions, visual taste, or domain ambiguity. When you ask, lead with what you already verified.

---

## Methodology

You are the General. The AI is the army.
You decide *what* and *why* — architecture, product, taste.
The AI executes — code, refactors, tests, plumbing.

### Feature workflow (5 stages, in order)

For every new feature, follow these five stages. Skipping a stage to "save time" almost always costs more time later.

#### 1. `grill-me` — clarify before you implement

Invoke `grill-me`. The skill interviews the user relentlessly until there's a shared understanding of what they actually want. One question at a time, each with a recommended answer. Use it when an idea has fuzzy edges or hidden assumptions.

#### 2. `to-prd` — synthesize into a GitHub issue

Once the interview converges, invoke `to-prd`. It synthesizes the conversation into a PRD using its template (Problem, Solution, User Stories, Implementation Decisions, Testing Decisions, Out of Scope) and publishes it as a GitHub issue with the `needs-triage` label.

PRDs live as GitHub issues, not as `.md` files. Issues don't drift. Files do.

#### 3. `to-issues` — break into vertical slices

Invoke `to-issues` against the PRD. It cuts the PRD into independently grabbable issues, each one a **vertical slice**: a thin path through DB → API → UI → tests, end to end.

Slice rules:
- Each slice is demoable on its own.
- Each slice is marked `HITL` (Human In The Loop) or `AFK` (autonomous).
- Prefer many thin slices over few thick ones.
- Never horizontal slices (all DB, then all API). Horizontal slices produce code that doesn't work end-to-end until the last day.

#### 4. `tdd` — red-green-refactor, vertically

For each issue, invoke `tdd`. The cycle:

```
RED   → write ONE test that confirms ONE behavior. It fails.
GREEN → write the minimal code to pass it.
RED   → next test, next behavior.
```

Never write all tests first. Tests written in bulk test imagined behavior, not real behavior — they pass when things break and break when they don't.

#### 5. `improve-codebase-architecture` — when the code feels muddy

Invoke periodically. It walks the codebase and proposes **deepening opportunities**: shallow modules to consolidate into deep ones (small interface, complex implementation hidden behind it).

It also maintains `CONTEXT.md` — every new domain concept that emerges goes there. This is the **ubiquitous language**: user, Claude, future sessions all speak the same words.

### Two bonus skills

- **`supabase-postgres-best-practices`** — invoke when touching SQL, indexes, RLS, or query performance.
- **`frontend-design`** — invoke **before** writing serious UI. Forces an aesthetic direction instead of dropping shadcn defaults everywhere. No Inter, no purple gradients, no "every dashboard looks the same."

### Ask the AI for / don't ask the AI for

| Ask the AI for | Don't ask the AI for |
|----------------|----------------------|
| Implement a feature | "Which architecture is better?" → that's the user's |
| Refactor a module | "What do users want?" → that's the user's |
| Write tests | Product decisions |
| Boilerplate, plumbing | Strong aesthetic calls |
| Verify (lint, build, test) | Domain intuition |

### LLMs hallucinate. Defend yourself.

- For **research questions**, append `use your search tool` to the prompt. Without it, the model invents references that don't exist.
- For **code questions**, prefer giving the AI the file to read over asking it to "remember" how a library works.
- A model's memory is compressed JPEG. Its tools are not.

---

## Documentation map

- [Architecture](./ARCHITECTURE.md) — system overview
- [Context](./CONTEXT.md) — domain vocabulary (maintained by `improve-codebase-architecture`)
- [Design](./docs/DESIGN.md) — design system, tokens, responsive
- [Frontend](./docs/FRONTEND.md) — TS conventions, components, i18n
- [Product sense](./docs/PRODUCT_SENSE.md) — PRD pointer, user stories
- [Quality](./docs/QUALITY_SCORE.md) — lint, typecheck, build, test gates
- [Reliability](./docs/RELIABILITY.md) — error handling, observability
- [Security](./docs/SECURITY.md) — RLS, auth, secrets, env vars
- [Plans](./docs/PLANS.md) — active and completed plans index
- [ADRs](./docs/adr/) — architectural decision records
- [Product specs](./docs/product-specs/index.md) — GitHub issue index

## Skills installed

| Skill                              | When to use it                                                          |
|------------------------------------|-------------------------------------------------------------------------|
| `grill-me`                         | Stress-test a plan or design before implementing                        |
| `tdd`                              | Drive a feature with red-green-refactor                                 |
| `improve-codebase-architecture`    | Find deep-module refactor opportunities; maintains `CONTEXT.md`         |
| `to-prd`                           | Synthesize current conversation into a PRD GitHub issue                 |
| `to-issues`                        | Break a PRD into vertical-slice GitHub issues                           |
| `supabase-postgres-best-practices` | Postgres schema, RLS, indexes, query optimization                       |
| `frontend-design`                  | Distinctive UI — typography, color, motion that isn't generic           |

Issue tracker: GitHub (use `gh` CLI). Triage label: `needs-triage`.

## Quickstart

\`\`\`bash
npm run dev
\`\`\`
```

### Étape 8.3 : Création de `ARCHITECTURE.md`

```markdown
# Architecture

System overview. High-level only — implementation details belong in `docs/`.

## Stack

| Layer        | Technology                          |
|--------------|-------------------------------------|
| Build        | Vite                                |
| Frontend     | React 18 + TypeScript               |
| Styling      | Tailwind CSS + shadcn/ui            |
| Routing      | React Router v6                     |
| State / data | TanStack Query                      |
| Forms        | React Hook Form + Zod               |
| i18n         | react-i18next (fr / en / de / lu)   |
| Backend      | Supabase (Postgres, Auth, Storage)  |

## Source layout

\`\`\`
src/
├── components/
│   ├── ui/              # shadcn/ui primitives — do not edit
│   ├── layout/
│   └── features/
├── hooks/
├── lib/
├── locales/             # fr / en / de / lu
├── pages/
├── services/            # API clients (supabase.ts...)
└── types/

supabase/
└── migrations/
\`\`\`

## Data flow

Browser → React component → TanStack Query → Supabase JS client → Supabase REST/Realtime → Postgres.

Auth state lives in `useAuth` hook subscribing to Supabase `onAuthStateChange`.

## Environment

| Variable                  | Description                       |
|---------------------------|-----------------------------------|
| `VITE_SUPABASE_URL`       | Supabase project URL              |
| `VITE_SUPABASE_ANON_KEY`  | Supabase public anon key          |
```

### Étape 8.4 : Création de `CONTEXT.md`

À la racine. Stub minimal — les concepts seront ajoutés par le skill `improve-codebase-architecture` au fil de l'eau.

```markdown
# Context

Domain vocabulary for this project. Every architectural decision and PRD uses these terms.

This file is **maintained by the `improve-codebase-architecture` skill**. When a new concept is named during a design conversation, add it here so the rest of the codebase (and Claude) speak the same language.

## Format

Each entry is a short, sharp definition:

\`\`\`markdown
## [Term]

One or two sentences defining the concept in this project's domain.
Optional: where it lives in the code, who owns it, what it is NOT.
\`\`\`

## Concepts

_(Empty for now. Add concepts as they emerge from design conversations.)_
```

### Étape 8.5 : Création des `docs/*.md`

#### `docs/DESIGN.md`

```markdown
# Design

## Responsive

Mobile-first. Build for `sm` and scale up.

| Breakpoint | Min width | Target          |
|------------|-----------|-----------------|
| `sm`       | 640px     | Large mobile    |
| `md`       | 768px     | Tablet          |
| `lg`       | 1024px    | Desktop         |
| `xl`       | 1280px    | Large desktop   |

Rules:
- No horizontal scroll at any breakpoint.
- Test on mobile, tablet, and desktop before shipping a UI change.
- Use Tailwind's `sm:`, `md:`, `lg:` prefixes — don't reinvent breakpoints.

## Design tokens

Defined as CSS variables in `src/index.css`. Light + dark themes share the same variable names.

Don't hardcode colors in components — reference the token (`bg-background`, `text-foreground`, `border-border`...).

## Aesthetic direction

Use the `frontend-design` skill before building UI. Reject generic AI aesthetics (Inter font, purple gradients, default shadcn-everywhere). Pick a deliberate aesthetic per screen and commit to it.

## SEO

- Each page sets a unique `<title>` and `<meta description>`.
- Use semantic HTML: `<header>`, `<main>`, `<nav>`, `<article>`.
- Images need a descriptive `alt`.
- URLs are clean and lowercase.
- Open Graph tags for shareable pages.
```

#### `docs/FRONTEND.md`

```markdown
# Frontend

## TypeScript

- Strict mode is on. Don't disable it.
- No `any` unless commented with the reason.
- Interfaces for component props. Types for API responses.

## Naming

| Kind        | Convention         | Example          |
|-------------|--------------------|------------------|
| Component   | PascalCase         | `UserProfile.tsx`|
| Hook        | `use` + camelCase  | `useAuth.ts`     |
| Utility     | camelCase          | `formatDate.ts`  |
| Type        | PascalCase         | `UserType`       |
| Constant    | UPPER_SNAKE_CASE   | `API_URL`        |

## Imports

Use the `@/` alias for everything inside `src/`:

\`\`\`ts
import { Button } from '@/components/ui/button'
import { useAuth } from '@/hooks/useAuth'
\`\`\`

## Component structure

\`\`\`tsx
// 1. Imports
import { useState } from 'react'

// 2. Types
interface Props { title: string }

// 3. Component
export function MyComponent({ title }: Props) {
  // 4. Hooks
  const [state, setState] = useState()

  // 5. Handlers
  const handleClick = () => {}

  // 6. Render
  return <div>{title}</div>
}
\`\`\`

One component per file. File name = component name.

## i18n (fr / en / de / lu)

Four languages, always in sync. `fr` is the fallback.

Rules:
- **No hardcoded strings** in components. Always `useTranslation()`.
- Translation keys in English, dot notation: `common.buttons.submit`.
- When you add a key, add it to all four locale files in the same commit.

Files live in `src/locales/<code>/translation.json`.

## Module depth

Prefer **deep modules**: a small, stable interface in front of complex implementation.

- The interface is what callers (and Claude) see — keep it tight.
- A wrapper that just forwards to one other call is a *shallow* module — delete it or deepen it.
- The `improve-codebase-architecture` skill is your friend when modules feel shallow.
```

#### `docs/PRODUCT_SENSE.md`

Ce fichier référence le PRD initial publié à la Phase 7 (GitHub issue). Substitue `[PRD_ISSUE_URL]` et `[PRD_ISSUE_NUMBER]` par les vrais valeurs récupérées à l'étape 7.3.

```markdown
# Product Sense

## Project description

[DESCRIPTION_COURTE_DU_PROJET — extraite du PRD]

## Initial PRD

The first PRD for this project was generated during init via `grill-me` + `to-prd`
and published as a GitHub issue:

- **Issue #[PRD_ISSUE_NUMBER]** — [PRD_ISSUE_URL]

Read it for the canonical statement of what the project is.

## PRD workflow (going forward)

PRDs live as **GitHub issues**. Don't keep parallel `.md` PRDs that drift.

1. Use `grill-me` to stress-test the idea.
2. Use `to-prd` to synthesize the conversation into a PRD GitHub issue.
3. Use `to-issues` to break that PRD into independently-grabbable vertical slices.

The issue tracker for this project is GitHub. The triage label is `needs-triage`.

See [`product-specs/index.md`](./product-specs/index.md) for the running index of PRDs.
```

#### `docs/QUALITY_SCORE.md`

```markdown
# Quality Score

Run these gates before saying "done."

## Commands

\`\`\`bash
npm run dev      # dev server
npm run build    # production build (also typechecks via `tsc -b`)
npm run lint     # ESLint
npm run preview  # preview the production build
\`\`\`

## Gates

| Gate          | Command            | When                         |
|---------------|--------------------|------------------------------|
| Typecheck     | `tsc --noEmit`     | After any TS change          |
| Lint          | `npm run lint`     | After any code change        |
| Build         | `npm run build`    | Before committing            |
| Manual UI     | open in Chrome     | After any UI change          |
| Tests         | `npm test`         | TDD cycles (when set up)     |

A change isn't done until every applicable gate is green. If the gate doesn't exist yet for a domain you're touching (e.g. tests for a new module), set it up — don't ship blind.
```

#### `docs/RELIABILITY.md`

```markdown
# Reliability

## Error handling

- Throw early, catch at the boundary (page / route handler).
- User-facing errors get a translated, actionable message — never a stack trace.
- Network errors: TanStack Query retries automatically; let it. Show a toast on permanent failure.
- Supabase errors: check `error` on every response, never assume `data` exists.

## Observability

- `console.error` for unexpected errors during development.
- TODO: wire up Sentry / equivalent before production traffic.

## Loading states

Every async operation has an explicit loading state. Never show a blank screen while data is in flight.

## Tests

The `tdd` skill enforces vertical-slice TDD (one test → one impl → repeat). Use it for non-trivial logic.

Tests verify **behavior through public interfaces**, not implementation details. If a test breaks during a refactor that didn't change behavior, the test was wrong, not the refactor.
```

#### `docs/SECURITY.md`

```markdown
# Security

## Secrets

- Secrets live in `.env`. Never commit `.env`.
- Frontend env vars are public — anything in `VITE_*` is shipped to the browser.
- Server-only secrets (service role keys, admin tokens) never go in `VITE_*`.

## Supabase

### Migrations

**Always** create a migration file before changing the database schema:

\`\`\`
supabase/migrations/YYYYMMDDHHMMSS_description.sql
\`\`\`

Example: `20250102143000_create_users_table.sql`

Never:
- Edit the schema via the dashboard without committing a matching migration.
- Use the SQL Editor for permanent changes.
- Apply changes outside the versioned migration flow.

The `supabase-postgres-best-practices` skill is your reference for schema, indexes, and RLS.

### Row Level Security (RLS)

- RLS is **on** for every table. No exceptions.
- Write explicit policies for every operation: SELECT, INSERT, UPDATE, DELETE.
- Test policies with different roles (anon, authenticated, service_role) before shipping.

### Auth

`useAuth` hook in `src/hooks/useAuth.ts` is the only entry point for auth state. Don't read `supabase.auth` directly from components.
```

#### `docs/PLANS.md`

```markdown
# Plans

Active and completed implementation plans. Each plan is a markdown file under `exec-plans/`.

## Active

See [`exec-plans/active/`](./exec-plans/active/).

## Completed

See [`exec-plans/completed/`](./exec-plans/completed/).

## Tech debt

See [`exec-plans/tech-debt-tracker.md`](./exec-plans/tech-debt-tracker.md).

## Format

Each plan file is named `YYYY-MM-DD-short-description.md` and contains:

- **Goal** — one sentence.
- **Why** — context the reader needs.
- **Approach** — the chosen design.
- **Steps** — checklist.
- **Risks / unknowns** — what could go wrong.
- **Status** — date + outcome on completion.

Move a plan to `completed/` when done. Don't delete; the history is useful.
```

#### `docs/exec-plans/tech-debt-tracker.md`

```markdown
# Tech debt tracker

A running list of known shortcuts, workarounds, and "fix later" decisions.

| Date       | Item | Why we punted | Fix when                    |
|------------|------|----------------|-----------------------------|
| YYYY-MM-DD | ...  | ...            | ...                         |

Add a row when you punt. Remove it when you fix it.
```

#### `docs/design-docs/index.md`

```markdown
# Design docs

Long-form design documents that explain the why behind a system or feature.

A design doc is appropriate when:
- A decision affects multiple modules.
- The reasoning would be hard to recover from the code alone.
- Future contributors need to understand the trade-offs to make compatible changes.

For single-decision records, use ADRs in [`../adr/`](../adr/).
```

#### `docs/adr/README.md`

```markdown
# Architectural Decision Records

One file per decision. Filename: `NNNN-short-title.md` (zero-padded, sequential).

Format:

\`\`\`markdown
# ADR NNNN: Short title

## Status
Accepted | Superseded by ADR-MMMM

## Context
What forced the decision. The constraints in play.

## Decision
What we chose.

## Consequences
What follows. What we're now ruling out.
\`\`\`

The `improve-codebase-architecture` skill may propose ADRs when a refactor candidate is rejected for a load-bearing reason.
```

#### `docs/product-specs/index.md`

Ajoute en première entrée le PRD créé à la Phase 7. Substitue `[PRD_ISSUE_NUMBER]`, `[PRD_ISSUE_URL]`, `[PRD_TITLE]` par les vraies valeurs.

```markdown
# Product specs

PRDs live as GitHub issues (created by the `to-prd` skill). This file indexes them.

## Active PRDs

- [PRD-[PRD_ISSUE_NUMBER]: [PRD_TITLE]]([PRD_ISSUE_URL])

## Completed

_(Move PRDs here once shipped.)_
```

#### `docs/references/README.md`

```markdown
# References

External library docs, `llms.txt` files, and any reference material useful to keep close to the code but not authored by us.

Add files here when an external library has documentation that benefits from being indexed alongside the codebase (e.g. `<lib>-llms.txt`).
```

#### `docs/generated/README.md`

```markdown
# Generated artifacts

Files in this folder are produced by tooling, not authored by hand.

Examples:
- `db-schema.md` — generated from the live Supabase schema.
- API specs from OpenAPI generation.

**Never edit files in this folder directly.** Re-run the generator and commit the new output.
```

### Étape 8.6 : Réinitialisation de `App.tsx`

```tsx
import { BrowserRouter, Routes, Route } from 'react-router-dom'
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'

const queryClient = new QueryClient()

function HomePage() {
  return (
    <div className="min-h-screen flex items-center justify-center">
      <div className="text-center">
        <h1 className="text-4xl font-bold mb-4">[NOM_PROJET]</h1>
        <p className="text-muted-foreground mb-8">[DESCRIPTION_COURTE_DU_PROJET]</p>
        <p className="text-sm text-muted-foreground">
          Prêt à développer ! Consulte CLAUDE.md pour les détails du projet.
        </p>
      </div>
    </div>
  )
}

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <BrowserRouter>
        <Routes>
          <Route path="/" element={<HomePage />} />
        </Routes>
      </BrowserRouter>
    </QueryClientProvider>
  )
}

export default App
```

---

## PHASE 9 : COMMIT + PUSH

Le repo GitHub existe déjà (créé en Phase 6) et le PRD y est publié comme issue (Phase 7). Il reste à committer et pusher tout le code + la doc avant de continuer sur la première feature.

### Étape 9.1 : Premier commit + push

```bash
git add .
git commit -m "Initial setup: [NOM_PROJET] (advanced) - skills, docs, PRD"
git branch -M main
git push -u origin main
```

### Étape 9.2 : Annonce de la suite

Dis :
```
✅ Code, doc et PRD pushés sur GitHub.

L'init technique est terminée. On continue MAINTENANT sur la première feature
en utilisant les skills installés. C'est volontaire : tu apprends la méthodo
en livrant du code réel, pas un toy example.

Workflow qui suit :
  Phase 10 → to-issues : on découpe le PRD en issues GitHub vertical-slice
  Phase 11 → tdd       : on livre la première issue en red-green-refactor
```

---

## PHASE 10 : DÉCOUPE DU PRD EN ISSUES (skill `to-issues`)

### Étape 10.1 : Invocation du skill `to-issues`

Récupère l'URL du PRD GitHub issue créé à la Phase 7 (étape 7.3). Tu l'as toujours en contexte.

**Invoque le skill `to-issues`** en lui passant la référence du PRD. Le skill va :
1. Lire le PRD GitHub issue.
2. Explorer le repo pour comprendre l'état actuel (scaffold React + Supabase, vide côté métier).
3. Proposer une liste numérotée de slices verticales (chaque slice traverse DB → API → UI → tests).
4. Marquer chaque slice `HITL` (besoin de l'humain) ou `AFK` (autonome).
5. Te quizzer sur la granularité, les dépendances, ce qui doit être HITL vs AFK.

Suis les instructions du skill exactement. Itère avec l'utilisateur jusqu'à approbation.

### Étape 10.2 : Publication des issues

Une fois le découpage approuvé, le skill publie chaque slice comme une nouvelle GitHub issue avec le label `needs-triage`, dans l'ordre des dépendances (les bloqueurs en premier pour pouvoir les référencer dans le champ "Blocked by").

Vérifie :
```bash
gh issue list --label needs-triage --json number,title,url
```

Tu dois voir le PRD parent + les N slices. Confirme à l'utilisateur :
```
PRD découpé en [N] issues. Voir le board : https://github.com/[USERNAME]/[NOM_PROJET]/issues

On attaque la première maintenant.
```

---

## PHASE 11 : PREMIÈRE FEATURE EN TDD (skill `tdd`)

### Étape 11.1 : Choix de la première issue

Liste les issues sans bloqueur (celles qui peuvent démarrer immédiatement) :

```bash
gh issue list --label needs-triage --json number,title,body --jq \
  '.[] | select(.body | contains("None - can start immediately")) | "#\(.number) \(.title)"'
```

Présente-les à l'utilisateur :
```
Voici les issues qu'on peut attaquer en premier (sans bloqueur) :

  #N  [TITRE_1]
  #M  [TITRE_2]
  ...

On commence par laquelle ? Si tu hésites, je recommande la plus simple ou
la plus "tracer bullet" (un slice mince qui prouve que la stack tient).
```

Attends le choix de l'utilisateur. Note le numéro + titre.

### Étape 11.2 : Branche dédiée

```bash
git checkout -b feature/issue-[NUMÉRO]-[slug-court]
```

### Étape 11.3 : Invocation du skill `tdd`

**Invoque le skill `tdd`**. Suis exactement son protocole :

1. **Planning** : Confirme avec l'utilisateur l'interface visée et les comportements à tester (priorités). Ne pas coder avant que la liste des comportements soit approuvée.
2. **Tracer bullet** : ÉCRIS UN seul test qui vérifie UN comportement. Lance-le → il échoue (RED). Écris le minimum de code pour le faire passer (GREEN).
3. **Boucle incrémentale** : pour chaque comportement restant — RED → GREEN, un à la fois. JAMAIS écrire tous les tests d'un coup.
4. **Refactor** : une fois tous les tests verts, cherche des opportunités de refactor (extraire la duplication, deepen modules). Lance les tests après chaque refactor.

Annonce chaque cycle à l'utilisateur :
```
🔴 RED   : test "[NOM_DU_TEST]"
🟢 GREEN : implémentation minimale
```

### Étape 11.4 : Vérification (Principe 2 : Verify before you ask)

Avant de demander à l'utilisateur de tester quoi que ce soit, tu lances :

```bash
npx tsc --noEmit
npm run lint
npm run build
```

Et si la slice touche l'UI, tu **ouvres l'app dans Chrome** (via le MCP Claude-in-Chrome si dispo) et tu fais le flow toi-même. Tu rapportes ce que tu as vérifié.

### Étape 11.5 : Commit + push de la feature

```bash
git add .
git commit -m "feat: [TITRE_FEATURE] (closes #[NUMÉRO])"
git push -u origin feature/issue-[NUMÉRO]-[slug-court]
```

Crée la PR :

```bash
gh pr create --title "[TITRE_FEATURE]" --body "Closes #[NUMÉRO]"
```

### Étape 11.6 : Récapitulatif final + handoff

Affiche :

```
🎉 [NOM_PROJET] : init terminée ET première feature livrée !

✅ Stack configurée                  (Phase 2)
✅ Supabase connecté                 (Phase 3-4)
✅ 7 skills installés                (Phase 5)
✅ Repo GitHub                       (Phase 6)
✅ PRD publié comme GitHub issue     (Phase 7)
✅ Documentation structurée          (Phase 8)
✅ Code initial pushé                (Phase 9)
✅ PRD découpé en issues             (Phase 10) — skill: to-issues
✅ Première feature livrée en TDD    (Phase 11) — skill: tdd

📁 Fichiers clés :
   CLAUDE.md             → principes + méthodologie + TOC + skills (lu chaque session)
   ARCHITECTURE.md       → vue système
   CONTEXT.md            → vocabulaire de domaine (alimenté par improve-codebase-architecture)
   docs/                 → DESIGN, FRONTEND, SECURITY, RELIABILITY, QUALITY_SCORE, PLANS

🔗 Repo  : https://github.com/[USERNAME]/[NOM_PROJET]
🔗 Issues: https://github.com/[USERNAME]/[NOM_PROJET]/issues
🔗 PR    : [URL_PR_CRÉÉE_À_L_ÉTAPE_11.5]

═══════════════════════════════════════════════════════════════════
SKILLS UTILISÉS PENDANT L'INIT (selon ton choix de Phase 7) :
  • Si tu as choisi le chemin grill-me :
      ✓ grill-me                   (Phase 7A — interview du PRD)
      ✓ to-prd                     (Phase 7A — PRD GitHub issue)
  • Si tu as collé ton PRD :
      (skills grill-me / to-prd pas utilisés cette fois,
       tu pourras les invoquer pour la prochaine feature)

  ✓ to-issues                      (Phase 10 — slices verticales)
  ✓ tdd                            (Phase 11 — red-green-refactor)
  ☐ improve-codebase-architecture  (à utiliser quand le code feel muddy)

Plus les 2 bonus :
  ☐ supabase-postgres-best-practices (à user quand tu touches au SQL)
  ☐ frontend-design                  (à user AVANT chaque écran sérieux)
═══════════════════════════════════════════════════════════════════

🚀 Prochaine feature
   Décris-moi ta prochaine idée. Je relance le cycle :
   grill-me → to-prd → to-issues → tdd.

   Ou attaque une autre issue déjà créée :
     gh issue list --label needs-triage
```

---

## NOTES IMPORTANTES

- **Exécute** toutes les commandes, ne te contente pas de les afficher.
- **Attends** la confirmation utilisateur pour les étapes critiques.
- **Adapte** le code selon les choix (avec/sans Supabase, avec/sans auth).
- **Le test de validation Supabase est OBLIGATOIRE** si Supabase est choisi.
- **Phase 7 = fork à demander à l'utilisateur** via AskUserQuestion : soit dogfood (`grill-me` + `to-prd`), soit paste depuis Cowork. Dans les deux cas le PRD finit comme GitHub issue avec label `needs-triage`. La suite (Phase 10 `to-issues`) marche pareil.
- **Phases 10 et 11 ne sont PAS optionnelles** : on enchaîne sur to-issues + tdd dès que le push initial est fait. C'est le coeur de la version advanced — tu sors avec une feature livrée et 4 skills déjà utilisés en pratique.
- **L'ordre des phases compte** : repo GitHub Phase 6 (avant `to-prd`), `to-prd` Phase 7 (avant `to-issues`), `to-issues` Phase 10 (avant `tdd`).
- **CLAUDE.md = principes + TOC**, pas une encyclopédie.
- **CONTEXT.md = stub au start**, alimenté par le skill `improve-codebase-architecture` au fil de l'eau.
- **La méthodologie complète vit dans `CLAUDE.md`** (section Methodology), pas dans un fichier séparé. Claude la lit à chaque session.
- **Migrations obligatoires** : toujours créer un fichier de migration avant de modifier Supabase.
- **Si une erreur survient**, explique clairement et propose une solution. Ne masque jamais un échec.
