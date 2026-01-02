# Initialisation de Projet - Guide Interactif

Tu es un assistant qui aide à créer un nouveau projet web moderne. Suis ce guide étape par étape de manière interactive.

## Étape 1 : Collecte d'informations

Pose ces questions à l'utilisateur UNE PAR UNE en utilisant le tool AskUserQuestion :

1. **Nom du projet** : "Comment veux-tu appeler ton projet ?" (ex: mon-app, dashboard-client)

2. **Description** : "Décris ton projet en une phrase"

3. **Backend Supabase** : "Tu as besoin d'un backend (base de données, auth, storage) ?"
   - Oui, j'ai besoin d'un backend
   - Non, juste du frontend

4. **Si backend = Oui**, demande :
   - "Tu veux l'authentification utilisateurs ?" (oui/non)
   - "Tu veux le storage de fichiers ?" (oui/non)

## Étape 2 : Setup du projet Vite + React + TypeScript

Une fois les infos collectées :

```bash
# Créer le projet dans un dossier temporaire
cd ..
npm create vite@latest [NOM_PROJET] -- --template react-ts
cd [NOM_PROJET]
```

## Étape 3 : Installation des dépendances

```bash
# Core
npm install

# Routing
npm install react-router-dom

# State & Data fetching
npm install @tanstack/react-query

# Forms
npm install react-hook-form zod @hookform/resolvers

# UI - Tailwind
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p

# UI - shadcn/ui
npm install tailwindcss-animate class-variance-authority clsx tailwind-merge
npm install lucide-react
npm install @radix-ui/react-slot
```

Si backend Supabase :
```bash
npm install @supabase/supabase-js
```

## Étape 4 : Configuration Tailwind

Remplace le contenu de `tailwind.config.js` :

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

## Étape 5 : Configuration CSS de base

Remplace le contenu de `src/index.css` :

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

## Étape 6 : Structure des dossiers

Crée cette structure :

```
src/
├── components/
│   └── ui/           # Composants shadcn/ui
├── hooks/            # Custom hooks
├── lib/
│   └── utils.ts      # Utilitaires (cn function)
├── pages/            # Pages de l'app
├── services/         # API calls, Supabase client
├── types/            # Types TypeScript
└── App.tsx
```

Crée `src/lib/utils.ts` :

```ts
import { type ClassValue, clsx } from "clsx"
import { twMerge } from "tailwind-merge"

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs))
}
```

## Étape 7 : Configuration Supabase (si demandé)

Si l'utilisateur veut Supabase :

1. Demande-lui : "Est-ce que tu as déjà créé ton projet Supabase ?"
   - Si non : "Va sur https://supabase.com, crée un compte et un nouveau projet. Reviens me voir quand c'est fait."
   - Si oui : Continue

2. Demande les clés :
   - "Quelle est l'URL de ton projet Supabase ?" (format: https://xxx.supabase.co)
   - "Quelle est ta clé anon/public ?" (commence par eyJ...)

3. Crée le fichier `.env` :

```env
VITE_SUPABASE_URL=https://xxx.supabase.co
VITE_SUPABASE_ANON_KEY=eyJ...
```

4. Crée `src/services/supabase.ts` :

```ts
import { createClient } from '@supabase/supabase-js'

const supabaseUrl = import.meta.env.VITE_SUPABASE_URL
const supabaseAnonKey = import.meta.env.VITE_SUPABASE_ANON_KEY

if (!supabaseUrl || !supabaseAnonKey) {
  throw new Error('Missing Supabase environment variables')
}

export const supabase = createClient(supabaseUrl, supabaseAnonKey)
```

5. Si auth demandée, crée `src/hooks/useAuth.ts` :

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

## Étape 8 : Configuration TypeScript paths

Mets à jour `tsconfig.json` pour ajouter les alias :

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

Et `vite.config.ts` :

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

Installe les types Node si besoin :
```bash
npm install -D @types/node
```

## Étape 9 : Setup MCP Supabase (si Supabase)

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
        "URL_SUPABASE",
        "--supabase-key",
        "CLE_SERVICE_ROLE"
      ]
    }
  }
}
```

Demande à l'utilisateur sa clé service_role (différente de anon key) pour le MCP.

## Étape 10 : Création du .gitignore

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

## Étape 11 : Création App.tsx de base

Remplace `src/App.tsx` :

```tsx
import { BrowserRouter, Routes, Route } from 'react-router-dom'
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'

const queryClient = new QueryClient()

function HomePage() {
  return (
    <div className="min-h-screen flex items-center justify-center">
      <div className="text-center">
        <h1 className="text-4xl font-bold mb-4">Bienvenue sur [NOM_PROJET]</h1>
        <p className="text-muted-foreground">[DESCRIPTION]</p>
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

## Étape 12 : Génération du README

Crée un `README.md` avec :

```markdown
# [NOM_PROJET]

[DESCRIPTION]

## Stack technique

- **Frontend** : React 18 + TypeScript + Vite
- **Styling** : Tailwind CSS + shadcn/ui
- **Routing** : React Router v6
- **State** : TanStack Query
- **Forms** : React Hook Form + Zod
[Si Supabase]
- **Backend** : Supabase (PostgreSQL, Auth, Storage)

## Installation

\`\`\`bash
npm install
\`\`\`

## Développement

\`\`\`bash
npm run dev
\`\`\`

## Build

\`\`\`bash
npm run build
\`\`\`

## Variables d'environnement

Copie `.env.example` vers `.env` et remplis les valeurs :

\`\`\`env
VITE_SUPABASE_URL=
VITE_SUPABASE_ANON_KEY=
\`\`\`
```

## Étape 13 : Création repo GitHub et push

Demande à l'utilisateur : "Tu veux que je crée le repo GitHub et push le code ?"

Si oui :

1. Vérifie que `gh` CLI est installé et authentifié :
```bash
gh auth status
```

2. Supprime le remote origin actuel (celui du template) :
```bash
git remote remove origin
```

3. Crée le repo et push :
```bash
gh repo create [NOM_PROJET] --public --source=. --remote=origin --push
```

Ou si privé :
```bash
gh repo create [NOM_PROJET] --private --source=. --remote=origin --push
```

## Étape 14 : Finalisation

1. Fais un premier commit propre :
```bash
git add .
git commit -m "Initial setup: React + TypeScript + Tailwind + shadcn/ui [+ Supabase]"
```

2. Affiche un récapitulatif à l'utilisateur :

```
Projet [NOM_PROJET] créé avec succès !

Stack installée :
- React 18 + TypeScript + Vite
- Tailwind CSS + shadcn/ui
- React Router v6
- TanStack Query
- React Hook Form + Zod
[Si Supabase] - Supabase (Auth + Storage)

Pour démarrer :
  cd [NOM_PROJET]
  npm run dev

Ton repo GitHub : https://github.com/[USERNAME]/[NOM_PROJET]
```

---

## Notes importantes

- Exécute TOUTES les commandes, ne te contente pas de les afficher
- Attends la confirmation de l'utilisateur pour les étapes critiques (création Supabase, push GitHub)
- Si une erreur survient, explique clairement le problème et propose une solution
- Adapte le code généré selon les choix de l'utilisateur (avec/sans Supabase, avec/sans auth, etc.)
