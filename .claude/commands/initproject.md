# Initialisation de Projet - Guide Interactif Complet

Tu es un assistant qui aide à créer un nouveau projet web moderne. Suis ce guide étape par étape de manière interactive.

---

## ÉTAPE 0 : DÉTECTION DE REPRISE (OBLIGATOIRE)

**AVANT TOUTE CHOSE**, vérifie l'état du projet en exécutant ces vérifications :

1. Vérifie si `package.json` existe (projet créé)
2. Vérifie si `.mcp.json` existe (Supabase MCP configuré)
3. Vérifie si `.env` existe (clés Supabase configurées)
4. Vérifie si `src/services/supabase.ts` existe (client Supabase créé)
5. Vérifie si `CLAUDE.md` existe (conventions générées)

**Règles de reprise :**

- Si **rien n'existe** → Commence à la PHASE 1
- Si **package.json existe mais pas .mcp.json** → Reprends à la PHASE 3 (config Supabase)
- Si **.mcp.json existe mais pas de table _quickstart_test** → Reprends à la PHASE 4 (test validation)
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

### Étape 2.3 : Configuration Tailwind

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

### Étape 2.4 : Configuration CSS

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

### Étape 2.5 : Structure des dossiers

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

### Étape 2.6 : Configuration i18n

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

### Étape 2.7 : Création du .gitignore

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

Demande les clés une par une :

1. "Quelle est l'URL de ton projet Supabase ?"
   - (Se trouve dans Settings > API, format: `https://xxx.supabase.co`)

2. "Quelle est ta clé anon/public ?"
   - (Se trouve dans Settings > API, commence par `eyJ...`)

3. "Quelle est ta clé service_role ?" (pour le MCP)
   - (Se trouve dans Settings > API, section "service_role key")

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

Crée le fichier `.mcp.json` à la racine :

```json
{
  "mcpServers": {
    "supabase": {
      "command": "npx",
      "args": [
        "-y",
        "@supabase/mcp-server-supabase@latest",
        "--supabase-url",
        "URL_SUPABASE_ICI",
        "--supabase-key",
        "CLE_SERVICE_ROLE_ICI"
      ]
    }
  }
}
```

Remplace les valeurs par celles récupérées à l'étape 3.2.

### Étape 3.7 : Rechargement de Claude Code pour activer le MCP

**IMPORTANT** : Le MCP Supabase est chargé au démarrage de Claude Code. Comme on vient de créer le `.mcp.json`, il faut relancer Claude Code.

Dis à l'utilisateur :
```
Le fichier .mcp.json est créé, mais Claude Code doit être relancé pour charger le MCP Supabase.

Fais ceci :
1. Quitte Claude Code (Ctrl+C ou tape "exit")
2. Relance Claude Code avec : claude
3. Tape /initproject pour reprendre

Je détecterai automatiquement que la Phase 3 est terminée et je passerai à la Phase 4 (test de validation).
```

**STOP ICI** - Attends que l'utilisateur relance Claude Code.

---

## PHASE 4 : TEST DE VALIDATION (si Supabase)

**IMPORTANT** : Cette phase vérifie que tout est bien configuré avant de commencer le vrai projet.

### Étape 4.0 : Détection de reprise

Au démarrage, vérifie si :
- Le fichier `.mcp.json` existe
- Le fichier `.env` existe avec les clés Supabase
- Le fichier `src/services/supabase.ts` existe

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

1. Le fichier .mcp.json existe-t-il ? ✓/✗
2. Les clés sont-elles correctes dans .mcp.json ?
3. As-tu bien relancé Claude Code après la création du .mcp.json ?

Si tu viens de créer le .mcp.json :
→ Quitte Claude Code (exit)
→ Relance : claude
→ Tape : /initproject

Si le problème persiste, vérifie que la service_role key est correcte dans .mcp.json
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

Le PRD ne va PAS dans le CLAUDE.md. Le CLAUDE.md contient uniquement les conventions génériques.

Le PRD sert à :
- Comprendre ce que l'utilisateur veut construire
- Pouvoir proposer des tables Supabase adaptées
- Savoir quelles pages/composants créer
- Guider le développement

Confirme à l'utilisateur :
```
J'ai bien compris ton projet. Voici ce que je retiens :

- [Résumé en 2-3 phrases]
- Principales fonctionnalités : [liste]
- Types d'utilisateurs : [liste]

Le CLAUDE.md avec les conventions génériques est prêt.
On peut commencer à développer quand tu veux !
```

---

## PHASE 6 : GÉNÉRATION DU CLAUDE.md

### Étape 6.1 : Création du CLAUDE.md

Crée le fichier `CLAUDE.md` à la racine du projet. Ce fichier contient les **conventions génériques** qui s'appliquent à TOUS les projets. Ne pas y mettre les specs du PRD.

```markdown
# Conventions Projet

## Supabase

- **Project Ref** : `[PROJECT_REF_SUPABASE]`
- **Dashboard** : https://supabase.com/dashboard/project/[PROJECT_REF_SUPABASE]

### Migrations (OBLIGATOIRE)

**TOUJOURS** créer un fichier de migration avant de modifier la base de données :

\`\`\`
supabase/migrations/
└── YYYYMMDDHHMMSS_description.sql
\`\`\`

Exemple : `20250102143000_create_users_table.sql`

**Ne JAMAIS** :
- Modifier la DB directement via le dashboard sans migration
- Utiliser le SQL Editor pour des changements permanents
- Appliquer des modifications sans fichier de migration versionné

**Process** :
1. Créer le fichier de migration dans `supabase/migrations/`
2. Écrire le SQL (CREATE, ALTER, etc.)
3. Appliquer via MCP ou `supabase db push`
4. Commit le fichier de migration

### Row Level Security (RLS)

- Activer RLS sur TOUTES les tables
- Créer des policies explicites pour chaque opération (SELECT, INSERT, UPDATE, DELETE)
- Tester les policies avec différents rôles

## Stack Technique

| Catégorie | Technologie |
|-----------|-------------|
| Build | Vite |
| Frontend | React 18 + TypeScript |
| Styling | Tailwind CSS + shadcn/ui |
| Routing | React Router v6 |
| State | TanStack Query |
| Forms | React Hook Form + Zod |
| Backend | Supabase |
| i18n | react-i18next |

## Internationalisation (i18n)

L'application doit supporter **4 langues** :

| Code | Langue |
|------|--------|
| `fr` | Français (défaut) |
| `en` | English |
| `de` | Deutsch |
| `lu` | Lëtzebuergesch |

### Structure des traductions

\`\`\`
src/locales/
├── fr/
│   └── translation.json
├── en/
│   └── translation.json
├── de/
│   └── translation.json
└── lu/
    └── translation.json
\`\`\`

### Règles

- **Aucun texte hardcodé** dans les composants
- Utiliser `useTranslation()` pour tous les textes
- Clés de traduction en anglais, format dot notation : `common.buttons.submit`
- Toujours ajouter les 4 langues en même temps

## Responsive Design

L'application doit être **mobile-first** et fonctionner sur :

| Breakpoint | Taille | Usage |
|------------|--------|-------|
| `sm` | 640px+ | Mobile large |
| `md` | 768px+ | Tablette |
| `lg` | 1024px+ | Desktop |
| `xl` | 1280px+ | Desktop large |

### Règles

- Commencer par le design mobile
- Utiliser les classes Tailwind responsive (`sm:`, `md:`, etc.)
- Tester sur mobile, tablette et desktop
- Pas de scroll horizontal

## SEO

### Règles

- Chaque page doit avoir un `<title>` unique et descriptif
- Utiliser les balises `<meta description>` appropriées
- Structure HTML sémantique (`<header>`, `<main>`, `<nav>`, `<article>`, etc.)
- Images avec attribut `alt` descriptif
- URLs propres et lisibles
- Balises Open Graph pour le partage social

### Composant SEO

Utiliser un composant `<SEO>` pour chaque page :

\`\`\`tsx
<SEO
  title="Titre de la page"
  description="Description pour les moteurs de recherche"
/>
\`\`\`

## Qualité du Code

### TypeScript

- **Strict mode** activé
- Pas de `any` sauf cas exceptionnels documentés
- Interfaces pour les props de composants
- Types pour les réponses API

### Conventions de nommage

| Type | Convention | Exemple |
|------|------------|---------|
| Composants | PascalCase | `UserProfile.tsx` |
| Hooks | camelCase + use | `useAuth.ts` |
| Utilitaires | camelCase | `formatDate.ts` |
| Types | PascalCase | `UserType` |
| Constantes | UPPER_SNAKE_CASE | `API_URL` |

### Imports

Toujours utiliser les alias `@/` :

\`\`\`ts
import { Button } from '@/components/ui/button'
import { useAuth } from '@/hooks/useAuth'
\`\`\`

### Structure des composants

\`\`\`tsx
// 1. Imports
import { useState } from 'react'

// 2. Types
interface Props {
  title: string
}

// 3. Composant
export function MyComponent({ title }: Props) {
  // 4. Hooks
  const [state, setState] = useState()

  // 5. Handlers
  const handleClick = () => {}

  // 6. Render
  return <div>{title}</div>
}
\`\`\`

### Fichiers

- Un composant par fichier
- Nom du fichier = nom du composant
- Index files pour les exports groupés

## Structure du Projet

\`\`\`
src/
├── components/
│   ├── ui/              # Composants shadcn/ui (ne pas modifier)
│   ├── layout/          # Header, Footer, Sidebar
│   └── features/        # Composants métier par feature
├── hooks/               # Custom hooks
├── lib/                 # Utilitaires
├── locales/             # Fichiers de traduction
├── pages/               # Pages/routes
├── services/            # API calls, Supabase client
└── types/               # Types TypeScript globaux
\`\`\`

## Commandes

\`\`\`bash
npm run dev      # Serveur de développement
npm run build    # Build production
npm run preview  # Preview du build
npm run lint     # Linter
\`\`\`

## Variables d'Environnement

| Variable | Description |
|----------|-------------|
| `VITE_SUPABASE_URL` | URL du projet Supabase |
| `VITE_SUPABASE_ANON_KEY` | Clé publique Supabase |

## Git

### Commits

Format : `type: description`

Types :
- `feat:` nouvelle fonctionnalité
- `fix:` correction de bug
- `refactor:` refactoring
- `style:` formatage, CSS
- `docs:` documentation
- `test:` tests
- `chore:` maintenance

### Branches

- `main` : production
- `develop` : développement
- `feature/xxx` : nouvelles features
- `fix/xxx` : corrections
```

### Étape 6.2 : Réinitialisation de App.tsx

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
        <p className="text-muted-foreground mb-8">[DESCRIPTION]</p>
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

### Étape 7.1 : Génération du README

Crée un `README.md` propre basé sur le PRD et le CLAUDE.md.

### Étape 7.2 : Création repo GitHub et push

Demande à l'utilisateur : "Tu veux que je crée le repo GitHub et push le code ?"

Si oui :

1. Supprime le remote origin du template :
```bash
git remote remove origin
```

2. Demande : "Le repo doit être public ou privé ?"

3. Crée le repo :
```bash
# Public
gh repo create [NOM_PROJET] --public --source=. --remote=origin --push

# Ou privé
gh repo create [NOM_PROJET] --private --source=. --remote=origin --push
```

### Étape 7.3 : Premier commit

```bash
git add .
git commit -m "Initial setup: [NOM_PROJET] - Ready to build"
git push
```

### Étape 7.4 : Récapitulatif final

Affiche :

```
🎉 Projet [NOM_PROJET] initialisé avec succès !

✅ Stack technique configurée (React, Tailwind, shadcn, i18n...)
✅ Supabase connecté et testé
✅ CLAUDE.md avec les conventions génériques
✅ Repo GitHub créé

📁 Fichiers importants :
   - CLAUDE.md           → Conventions (SEO, i18n, migrations, etc.)
   - .env                → Variables d'environnement
   - .mcp.json           → Configuration MCP Supabase
   - supabase/migrations → Fichiers de migration SQL

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
- **Le CLAUDE.md contient les conventions génériques**, pas les specs du projet
- **Le PRD sert uniquement à comprendre** ce que l'utilisateur veut construire
- **Migrations obligatoires** : toujours créer un fichier de migration avant de modifier Supabase
- Si une erreur survient, explique clairement et propose une solution
