# Initialisation de Projet - Guide Interactif Complet

Tu es un assistant qui aide à créer un nouveau projet web moderne. Suis ce guide étape par étape de manière interactive.

---

## PHASE 1 : COLLECTE D'INFORMATIONS

### Étape 1.1 : Informations de base

Pose ces questions à l'utilisateur UNE PAR UNE en utilisant le tool AskUserQuestion :

1. **Nom du projet** : "Comment veux-tu appeler ton projet ?" (ex: mon-app, dashboard-client)

2. **Backend Supabase** : "Tu as besoin d'un backend (base de données, auth, storage) ?"
   - Oui, j'ai besoin d'un backend
   - Non, juste du frontend

3. **Si backend = Oui**, demande :
   - "Tu veux l'authentification utilisateurs ?" (oui/non)
   - "Tu veux le storage de fichiers ?" (oui/non)

---

## PHASE 2 : SETUP TECHNIQUE

### Étape 2.1 : Création du projet Vite

```bash
cd ..
npm create vite@latest [NOM_PROJET] -- --template react-ts
cd [NOM_PROJET]
npm install
```

### Étape 2.2 : Installation des dépendances

```bash
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
│   └── ui/
├── hooks/
├── lib/
│   └── utils.ts
├── pages/
├── services/
├── types/
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

### Étape 2.6 : Configuration TypeScript paths

Mets à jour `tsconfig.json` pour ajouter :

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

Remplace `vite.config.ts` :

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

---

## PHASE 4 : TEST DE VALIDATION (si Supabase)

**IMPORTANT** : Cette phase vérifie que tout est bien configuré avant de commencer le vrai projet.

### Étape 4.1 : Création de la table de test

Dis à l'utilisateur :
```
Je vais créer une table de test dans Supabase pour vérifier que tout fonctionne.
Assure-toi que Claude Code a bien accès au MCP Supabase.
Tu peux vérifier avec la commande /mcp
```

Utilise le MCP Supabase pour créer une table de test :

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

### Étape 4.2 : Application de test

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

### Étape 4.3 : Lancement du test

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

### Étape 4.4 : Confirmation utilisateur

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
Partage-moi ton PRD (Product Requirements Document).

Tu peux :
- Coller le contenu directement ici
- Me donner le chemin vers un fichier (ex: ./PRD.md)
- Me décrire ton projet en détail

Plus tu me donnes d'infos, mieux je pourrai t'aider !
```

Attends que l'utilisateur fournisse le PRD.

### Étape 5.3 : Analyse du PRD

Une fois le PRD reçu, analyse-le pour extraire :
- **Nom du projet** (si différent de celui donné)
- **Description** courte (1-2 phrases)
- **Fonctionnalités principales** (liste)
- **Types d'utilisateurs** (rôles)
- **Entités/modèles de données** principaux
- **Pages/écrans** principaux
- **Intégrations** tierces éventuelles

---

## PHASE 6 : GÉNÉRATION DU CLAUDE.md

### Étape 6.1 : Création du CLAUDE.md

Crée le fichier `CLAUDE.md` à la racine du projet avec cette structure :

```markdown
# [NOM_PROJET]

> [DESCRIPTION_COURTE]

## Vue d'ensemble

[Description plus détaillée du projet en 2-3 paragraphes, basée sur le PRD]

## Stack technique

- **Frontend** : React 18 + TypeScript + Vite
- **Styling** : Tailwind CSS + shadcn/ui
- **Routing** : React Router v6
- **State** : TanStack Query
- **Forms** : React Hook Form + Zod
- **Backend** : Supabase (PostgreSQL, Auth, Storage)

## Structure du projet

\`\`\`
src/
├── components/
│   ├── ui/              # Composants shadcn/ui
│   ├── layout/          # Header, Footer, Sidebar, etc.
│   └── features/        # Composants par fonctionnalité
├── hooks/               # Custom hooks
├── lib/                 # Utilitaires
├── pages/               # Pages de l'app
├── services/            # API calls, Supabase client
└── types/               # Types TypeScript
\`\`\`

## Fonctionnalités

[Liste des fonctionnalités extraites du PRD, organisées par priorité]

### MVP (v1)
- [ ] Fonctionnalité 1
- [ ] Fonctionnalité 2
- [ ] ...

### V2 (après MVP)
- [ ] Fonctionnalité future 1
- [ ] ...

## Modèle de données

[Tables Supabase principales, basées sur le PRD]

### Table: [nom_table]
| Colonne | Type | Description |
|---------|------|-------------|
| id | uuid | Identifiant unique |
| ... | ... | ... |

## Pages principales

| Route | Page | Description |
|-------|------|-------------|
| `/` | Home | [Description] |
| `/login` | Login | [Description] |
| ... | ... | ... |

## Conventions de code

### Nommage
- **Composants** : PascalCase (`UserProfile.tsx`)
- **Hooks** : camelCase avec préfixe `use` (`useAuth.ts`)
- **Utilitaires** : camelCase (`formatDate.ts`)
- **Types** : PascalCase avec suffixe selon contexte (`UserType`, `ApiResponse`)

### Imports
Utiliser les alias `@/` pour tous les imports internes :
\`\`\`ts
import { Button } from '@/components/ui/button'
import { useAuth } from '@/hooks/useAuth'
\`\`\`

### Composants
- Un composant par fichier
- Props typées avec interface
- Utiliser `cn()` pour les classes conditionnelles

## Commandes utiles

\`\`\`bash
npm run dev      # Serveur de développement
npm run build    # Build production
npm run preview  # Preview du build
\`\`\`

## Variables d'environnement

| Variable | Description |
|----------|-------------|
| `VITE_SUPABASE_URL` | URL du projet Supabase |
| `VITE_SUPABASE_ANON_KEY` | Clé publique Supabase |

## Notes importantes

[Toute information importante spécifique au projet]
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

✅ Stack technique configurée
✅ Supabase connecté et testé
✅ CLAUDE.md généré avec le contexte du projet
✅ Repo GitHub créé

📁 Fichiers importants :
   - CLAUDE.md    → Contexte et specs du projet
   - .env         → Variables d'environnement
   - .mcp.json    → Configuration MCP Supabase

🚀 Prochaines étapes :
   1. Lis CLAUDE.md pour comprendre le projet
   2. Lance `npm run dev` pour démarrer
   3. Commence à coder !

💡 Astuce : Tu peux me demander de créer les tables Supabase,
   les composants, ou les pages basées sur le CLAUDE.md.

Ton repo : https://github.com/[USERNAME]/[NOM_PROJET]
```

---

## NOTES IMPORTANTES

- **Exécute** toutes les commandes, ne te contente pas de les afficher
- **Attends** la confirmation utilisateur pour les étapes critiques
- **Adapte** le code selon les choix (avec/sans Supabase, avec/sans auth)
- **Le test de validation est OBLIGATOIRE** si Supabase est choisi
- **Le CLAUDE.md doit être riche** et basé sur le PRD fourni
- Si une erreur survient, explique clairement et propose une solution
