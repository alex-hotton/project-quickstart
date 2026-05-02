# Initialisation de Projet - Guide Interactif Complet

Tu es un assistant qui aide à créer un nouveau projet web moderne. Suis ce guide étape par étape de manière interactive.

---

## ÉTAPE 0 : DÉTECTION DE REPRISE (OBLIGATOIRE)

**AVANT TOUTE CHOSE**, vérifie l'état du projet en exécutant ces vérifications :

1. Vérifie si `package.json` existe (projet créé)
2. Vérifie si `.env` existe (clés Supabase configurées)
3. Vérifie si `src/services/supabase.ts` existe (client Supabase créé)
4. Vérifie si `CLAUDE.md` existe (conventions générées)
5. Vérifie si le MCP Supabase est disponible (essaie d'utiliser un outil MCP Supabase)

**Règles de reprise :**

- Si **rien n'existe** → Commence à la PHASE 1
- Si **package.json existe mais pas .env** → Reprends à la PHASE 3 (config Supabase)
- Si **.env existe et MCP disponible mais pas de table _quickstart_test** → Reprends à la PHASE 4 (test validation)
- Si **CLAUDE.md existe** → Tout est déjà configuré, dis à l'utilisateur que le projet est prêt

**EXÉCUTE CES VÉRIFICATIONS MAINTENANT** avant de continuer. Utilise les outils Glob ou Read pour vérifier l'existence des fichiers.

---

## PHASE 1 : COLLECTE D'INFORMATIONS

### Étape 1.1 : Détection du nom du projet

Le nom du projet est le nom du dossier courant (celui dans lequel on travaille).

Utilise `pwd` ou regarde le chemin pour extraire le nom du dossier.

Par exemple si le chemin est `/Users/alex/projects/mon-super-projet`, le nom du projet est `mon-super-projet`.

Confirme à l'utilisateur :
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

**IMPORTANT** : On travaille dans le dossier COURANT (celui cloné depuis project-quickstart).

D'abord, supprime les fichiers template qui ne sont plus nécessaires :

```bash
rm -rf templates/
rm -f README.md
```

Garde le dossier `.claude/commands/` car il contient ce script.

### Étape 2.2 : Création du projet (manuel)

**NOTE** : On ne peut PAS utiliser `npm create vite` car le dossier n'est pas vide (il contient .git et .claude).
On crée donc le projet manuellement.

Crée le fichier `package.json` :

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

Crée le fichier `index.html` à la racine :

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

Crée `src/App.tsx` (temporaire, sera remplacé plus tard) :

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
# Core Vite + React + TypeScript
npm install react react-dom
npm install -D vite @vitejs/plugin-react typescript @types/react @types/react-dom

# Routing
npm install react-router-dom

# State & Data fetching
npm install @tanstack/react-query

# Forms
npm install react-hook-form zod @hookform/resolvers

# Internationalization (i18n)
npm install react-i18next i18next i18next-browser-languagedetector

# UI - Tailwind
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p

# UI - shadcn/ui
npm install tailwindcss-animate class-variance-authority clsx tailwind-merge
npm install lucide-react
npm install @radix-ui/react-slot

# Types Node
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
      screens: {
        "2xl": "1400px",
      },
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

Crée cette structure :

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
│   ├── fr/
│   │   └── translation.json
│   ├── en/
│   │   └── translation.json
│   ├── de/
│   │   └── translation.json
│   └── lu/
│       └── translation.json
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
    interpolation: {
      escapeValue: false,
    },
  })

export default i18n
```

Crée les fichiers de traduction de base :

`src/locales/fr/translation.json` :
```json
{
  "common": {
    "loading": "Chargement...",
    "error": "Une erreur est survenue",
    "save": "Enregistrer",
    "cancel": "Annuler",
    "delete": "Supprimer",
    "edit": "Modifier",
    "back": "Retour"
  }
}
```

`src/locales/en/translation.json` :
```json
{
  "common": {
    "loading": "Loading...",
    "error": "An error occurred",
    "save": "Save",
    "cancel": "Cancel",
    "delete": "Delete",
    "edit": "Edit",
    "back": "Back"
  }
}
```

`src/locales/de/translation.json` :
```json
{
  "common": {
    "loading": "Laden...",
    "error": "Ein Fehler ist aufgetreten",
    "save": "Speichern",
    "cancel": "Abbrechen",
    "delete": "Löschen",
    "edit": "Bearbeiten",
    "back": "Zurück"
  }
}
```

`src/locales/lu/translation.json` :
```json
{
  "common": {
    "loading": "Lueden...",
    "error": "E Feeler ass opgetrueden",
    "save": "Späicheren",
    "cancel": "Ofbriechen",
    "delete": "Läschen",
    "edit": "Änneren",
    "back": "Zréck"
  }
}
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

Attends que l'utilisateur fournisse les 2 valeurs.

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

Si l'utilisateur veut l'auth, crée `src/hooks/useAuth.ts` :

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

### Étape 3.6 : Configuration MCP Supabase

Dis à l'utilisateur :
```
Maintenant on va connecter le MCP Supabase à Claude Code.

Fais ceci :
1. Quitte Claude Code avec : /exit
2. Dans ton terminal, exécute cette commande :

   claude mcp add --scope project --transport http supabase "https://mcp.supabase.com/mcp"

3. Relance Claude Code avec : claude --dangerously-skip-permissions
4. Tape : /mcp
5. Valide l'authentification Supabase dans le navigateur quand on te le demande
   (une fenêtre va s'ouvrir automatiquement pour te connecter à Supabase)
6. Tape /initproject pour reprendre

Je détecterai automatiquement que la Phase 3 est terminée et je passerai à la Phase 4 (test de validation).
```

**STOP ICI** - Attends que l'utilisateur relance Claude Code.

---

## PHASE 4 : TEST DE VALIDATION (si Supabase)

**IMPORTANT** : Cette phase vérifie que tout est bien configuré avant de commencer le vrai projet.

### Étape 4.0 : Détection de reprise

Au démarrage, vérifie si :
- Le fichier `.env` existe avec les clés Supabase
- Le fichier `src/services/supabase.ts` existe
- Le MCP Supabase est disponible (essaie d'utiliser un outil MCP)

Si OUI → La Phase 3 est terminée, passe directement à l'étape 4.1

Si NON → Reprends depuis la Phase 1

### Étape 4.1 : Vérification du MCP Supabase

**AVANT de créer la table de test**, vérifie que le MCP Supabase est bien chargé.

Dis à l'utilisateur :
```
Je vais vérifier que le MCP Supabase est bien configuré.
```

Ensuite, essaie d'utiliser le MCP Supabase (par exemple, liste les tables existantes ou exécute une requête simple).

**Si le MCP fonctionne** → Continue à l'étape 4.2

**Si le MCP ne fonctionne pas** (erreur, timeout, pas de réponse) :

Dis à l'utilisateur :
```
Le MCP Supabase n'est pas accessible. Vérifions :

1. As-tu bien exécuté la commande ?
   → claude mcp add --scope project --transport http supabase "https://mcp.supabase.com/mcp"
2. As-tu bien relancé Claude Code après ?
3. As-tu validé l'authentification Supabase dans le navigateur via /mcp ?

Si ce n'est pas encore fait :
→ Quitte Claude Code : /exit
→ Exécute : claude mcp add --scope project --transport http supabase "https://mcp.supabase.com/mcp"
→ Relance : claude --dangerously-skip-permissions
→ Tape : /mcp et valide l'authentification Supabase dans le navigateur
→ Tape : /initproject
```

**STOP** - Attends que l'utilisateur corrige et relance.

### Étape 4.2 : Création de la table de test

Utilise le MCP Supabase pour exécuter ce SQL :

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

-- Activer RLS mais autoriser la lecture publique pour le test
ALTER TABLE _quickstart_test ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Allow public read" ON _quickstart_test FOR SELECT USING (true);
```

Confirme à l'utilisateur :
```
✅ Table de test créée dans Supabase !
```

### Étape 4.3 : Application de test

Remplace `src/App.tsx` avec une app de test :

```tsx
import { useEffect, useState } from 'react'
import { supabase } from '@/services/supabase'

interface TestRow {
  id: number
  message: string
  created_at: string
}

function App() {
  const [data, setData] = useState<TestRow[]>([])
  const [loading, setLoading] = useState(true)
  const [error, setError] = useState<string | null>(null)

  useEffect(() => {
    async function fetchData() {
      const { data, error } = await supabase
        .from('_quickstart_test')
        .select('*')
        .order('id')

      if (error) {
        setError(error.message)
      } else {
        setData(data || [])
      }
      setLoading(false)
    }
    fetchData()
  }, [])

  return (
    <div className="min-h-screen bg-background flex items-center justify-center p-8">
      <div className="max-w-md w-full">
        <div className="bg-card border rounded-lg p-6 shadow-sm">
          <h1 className="text-2xl font-bold text-center mb-6">
            🚀 Test de Configuration
          </h1>

          {loading && (
            <div className="text-center text-muted-foreground">
              Chargement...
            </div>
          )}

          {error && (
            <div className="bg-destructive/10 border border-destructive rounded p-4">
              <p className="text-destructive font-medium">❌ Erreur de connexion</p>
              <p className="text-sm text-destructive/80 mt-1">{error}</p>
              <p className="text-sm mt-2">Vérifie tes variables d'environnement dans .env</p>
            </div>
          )}

          {!loading && !error && data.length > 0 && (
            <div className="space-y-4">
              <div className="bg-green-500/10 border border-green-500 rounded p-4">
                <p className="text-green-600 font-medium">✅ Connexion Supabase réussie !</p>
              </div>

              <div className="border rounded">
                <table className="w-full">
                  <thead className="bg-muted">
                    <tr>
                      <th className="text-left p-2 text-sm font-medium">ID</th>
                      <th className="text-left p-2 text-sm font-medium">Message</th>
                    </tr>
                  </thead>
                  <tbody>
                    {data.map((row) => (
                      <tr key={row.id} className="border-t">
                        <td className="p-2 text-sm">{row.id}</td>
                        <td className="p-2 text-sm">{row.message}</td>
                      </tr>
                    ))}
                  </tbody>
                </table>
              </div>

              <p className="text-center text-sm text-muted-foreground">
                Si tu vois ce tableau, tout est bien configuré !
              </p>
            </div>
          )}

          {!loading && !error && data.length === 0 && (
            <div className="bg-yellow-500/10 border border-yellow-500 rounded p-4">
              <p className="text-yellow-600 font-medium">⚠️ Table vide</p>
              <p className="text-sm mt-1">La connexion fonctionne mais la table de test est vide.</p>
            </div>
          )}
        </div>
      </div>
    </div>
  )
}

export default App
```

### Étape 4.4 : Lancement du test

Dis à l'utilisateur :
```
Je lance le serveur de développement.
Ouvre http://localhost:5173 dans ton navigateur.

Tu devrais voir :
✅ Un message "Connexion Supabase réussie !"
✅ Un tableau avec 3 lignes de test

Dis-moi si ça fonctionne !
```

Lance le serveur :
```bash
npm run dev
```

### Étape 4.5 : Confirmation utilisateur

Demande à l'utilisateur : "Est-ce que tu vois le tableau de test avec les 3 messages ?"

- Si **oui** : Continue vers la Phase 5
- Si **non** :
  - Demande quel est le message d'erreur affiché
  - Aide à diagnostiquer (mauvaises clés, RLS, etc.)
  - Recommence le test

---

## PHASE 5 : NETTOYAGE ET PRD

### Étape 5.1 : Suppression de la table de test

Une fois le test validé, supprime la table de test via MCP :

```sql
DROP TABLE IF EXISTS _quickstart_test;
```

### Étape 5.2 : Demande du PRD

Dis à l'utilisateur :
```
Parfait ! Tout est bien configuré. 🎉

Maintenant, j'ai besoin de comprendre ton projet.
Partage-moi ton PRD (Product Requirements Document) ou décris-moi ton application.

Tu peux :
- Coller le contenu directement ici
- Me donner le chemin vers un fichier (ex: ./PRD.md)
- Me décrire ton projet en détail

C'est juste pour que je comprenne ce qu'on va construire ensemble.
Plus tu me donnes d'infos, mieux je pourrai t'aider !
```

Attends que l'utilisateur fournisse le PRD.

### Étape 5.3 : Analyse du PRD

Une fois le PRD reçu, **LIS-LE et COMPRENDS-LE** pour pouvoir aider l'utilisateur par la suite.

Extrait :
- **Une description courte** (1-2 phrases) — elle ira en haut du `CLAUDE.md`
- Les fonctionnalités principales
- Les types d'utilisateurs

Confirme à l'utilisateur :
```
J'ai bien compris ton projet. Voici ce que je retiens :

- [Résumé en 2-3 phrases]
- Principales fonctionnalités : [liste]
- Types d'utilisateurs : [liste]

Je prépare maintenant la documentation du projet.
```

---

## PHASE 6 : GÉNÉRATION DE LA DOCUMENTATION

La doc suit la structure **OpenAI Harness Engineering** : `CLAUDE.md` à la racine est une table des matières, le contenu réel vit dans `docs/*.md` thématiques. Pas de mur de texte dans CLAUDE.md.

### Étape 6.1 : Création de l'arborescence docs

Crée la structure complète :

```bash
mkdir -p docs/design-docs docs/exec-plans/active docs/exec-plans/completed docs/generated docs/product-specs docs/references docs/adr
```

### Étape 6.2 : Création de `CLAUDE.md`

Crée le fichier `CLAUDE.md` à la racine :

```markdown
# [NOM_PROJET]

[DESCRIPTION_COURTE_DU_PROJET — 1-2 lignes extraites du PRD]

---

## Principles

### 1. Documentation is the source of truth

Update docs in the same commit as the code. A change without a doc update is incomplete.

- `CLAUDE.md` is principles + table of contents. Nothing else.
- `ARCHITECTURE.md` is the system at a glance.
- One topic per file in `docs/`. Lead with the why, then the how.
- Short sections, concrete examples, no padding.
- Decisions go in `docs/adr/`. Active plans in `docs/exec-plans/active/`, archived to `completed/` when done.
- Generated artifacts in `docs/generated/` are regenerated, never hand-edited.
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

## Documentation map

- [Architecture](./ARCHITECTURE.md) — system overview
- [Design](./docs/DESIGN.md) — design system, tokens, responsive
- [Frontend](./docs/FRONTEND.md) — TS conventions, components, i18n
- [Product sense](./docs/PRODUCT_SENSE.md) — PRD, user stories
- [Quality](./docs/QUALITY_SCORE.md) — lint, typecheck, build, test gates
- [Reliability](./docs/RELIABILITY.md) — error handling, observability
- [Security](./docs/SECURITY.md) — RLS, auth, secrets, env vars
- [Plans](./docs/PLANS.md) — active and completed plans index
- [ADRs](./docs/adr/) — architectural decision records

## Quickstart

\`\`\`bash
npm run dev
\`\`\`
```

### Étape 6.3 : Création de `ARCHITECTURE.md`

Crée à la racine :

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
│   ├── layout/          # Header, Footer, Sidebar
│   └── features/        # feature-scoped components
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

### Étape 6.4 : Création des fichiers `docs/*.md`

Crée chaque fichier avec le contenu indiqué.

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

## SEO

- Each page sets a unique `<title>` and `<meta description>`.
- Use semantic HTML: `<header>`, `<main>`, `<nav>`, `<article>`.
- Images need a descriptive `alt`.
- URLs are clean and lowercase.
- Open Graph tags for shareable pages.

Use the `<SEO>` component (when created):

\`\`\`tsx
<SEO title="Page title" description="..." />
\`\`\`
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

| Code | Language        |
|------|-----------------|
| `fr` | Français        |
| `en` | English         |
| `de` | Deutsch         |
| `lu` | Lëtzebuergesch  |

Rules:
- **No hardcoded strings** in components. Always `useTranslation()`.
- Translation keys in English, dot notation: `common.buttons.submit`.
- When you add a key, add it to all four locale files in the same commit.

Files live in `src/locales/<code>/translation.json`.
```

#### `docs/PRODUCT_SENSE.md`

```markdown
# Product Sense

## Project description

[DESCRIPTION_COURTE_DU_PROJET]

## Main features

[LISTE_FONCTIONNALITES_DU_PRD]

## User types

[LISTE_TYPES_UTILISATEURS]

## Source PRD

The full PRD lives in `docs/product-specs/`. Product specs are the source of truth for what to build.
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

A change isn't done until every applicable gate is green.
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

Process:
1. Write the SQL in `supabase/migrations/`.
2. Apply via MCP or `supabase db push`.
3. Commit the migration file.

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

Format:

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

#### `docs/product-specs/index.md`

```markdown
# Product specs

The source PRD and feature specs live here.

`PRD.md` is the canonical statement of what this product does and why. Update it when scope changes.
```

Si l'user a fourni un PRD, sauvegarde-le dans `docs/product-specs/PRD.md`.

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

### Étape 6.5 : Réinitialisation de `App.tsx`

Remplace `src/App.tsx` avec une version propre pour démarrer :

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

## PHASE 7 : FINALISATION

### Étape 7.1 : Création repo GitHub et push

Demande à l'utilisateur : "Tu veux que je crée le repo GitHub et push le code ?"

Si oui :

1. Supprime le remote origin du template :
```bash
git remote remove origin
```

2. Crée le repo (toujours en privé par défaut) :
```bash
gh repo create [NOM_PROJET] --private --source=. --remote=origin --push
```

### Étape 7.2 : Premier commit

```bash
git add .
git commit -m "Initial setup: [NOM_PROJET] - Ready to build"
git push
```

### Étape 7.3 : Récapitulatif final

Affiche :

```
🎉 Projet [NOM_PROJET] initialisé avec succès !

✅ Stack technique configurée (React, Tailwind, shadcn, i18n...)
✅ Supabase connecté et testé
✅ Documentation structurée (CLAUDE.md + docs/)
✅ Repo GitHub créé

📁 Fichiers importants :
   - CLAUDE.md           → Principes + table des matières
   - ARCHITECTURE.md     → Vue système
   - docs/               → Documentation thématique
   - .env                → Variables d'environnement
   - supabase/migrations → Fichiers de migration SQL

🔌 MCP Supabase connecté via : claude mcp add

🌍 Langues configurées : FR, EN, DE, LU

🚀 Prochaines étapes :
   1. Lance `npm run dev` pour démarrer
   2. Demande-moi de créer les tables Supabase
   3. Demande-moi de créer les pages/composants

💡 Rappel : Pour toute modification de la DB,
   je créerai d'abord un fichier de migration.

Ton repo : https://github.com/[USERNAME]/[NOM_PROJET]
```

---

## NOTES IMPORTANTES

- **Exécute** toutes les commandes, ne te contente pas de les afficher
- **Attends** la confirmation utilisateur pour les étapes critiques
- **Adapte** le code selon les choix (avec/sans Supabase, avec/sans auth)
- **Le test de validation est OBLIGATOIRE** si Supabase est choisi
- **CLAUDE.md est une table des matières + principes**, pas une encyclopédie
- **Le contenu thématique vit dans `docs/*.md`** — splitte, ne concatène pas
- **Migrations obligatoires** : toujours créer un fichier de migration avant de modifier Supabase
- Si une erreur survient, explique clairement et propose une solution
