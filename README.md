# Project Quickstart

Crée un projet web moderne en 2 minutes avec Claude Code.

## Stack incluse

| Catégorie | Technologies |
|-----------|-------------|
| **Build** | Vite |
| **Frontend** | React 18 + TypeScript |
| **Styling** | Tailwind CSS + shadcn/ui |
| **Routing** | React Router v6 |
| **Data** | TanStack Query |
| **Forms** | React Hook Form + Zod |
| **Backend** | Supabase (optionnel) |

## Prérequis

- [Node.js](https://nodejs.org/) 18+
- [Claude Code](https://claude.com/code) installé
- [GitHub CLI](https://cli.github.com/) (optionnel, pour créer le repo automatiquement)

## Utilisation

### 1. Clone ce repo

```bash
git clone https://github.com/alex-hotton/project-quickstart.git mon-projet
cd mon-projet
```

### 2. Lance Claude Code

```bash
claude
```

### 3. Exécute la commande d'initialisation

```
/init
```

### 4. Réponds aux questions

Claude Code va te demander :
- Le nom de ton projet
- Une description
- Si tu veux un backend Supabase
- Si tu veux l'authentification
- Si tu veux le storage fichiers

### 5. C'est prêt !

Claude Code va :
- Installer toutes les dépendances
- Configurer Tailwind + shadcn/ui
- Créer la structure de dossiers
- Configurer Supabase (si demandé)
- Créer le repo GitHub
- Push le premier commit

## Ce que tu obtiens

```
mon-projet/
├── src/
│   ├── components/
│   │   └── ui/           # Composants shadcn/ui
│   ├── hooks/            # Custom hooks (+ useAuth si Supabase)
│   ├── lib/
│   │   └── utils.ts      # Utilitaires
│   ├── pages/            # Pages de l'app
│   ├── services/         # Supabase client
│   ├── types/            # Types TypeScript
│   └── App.tsx
├── .env                  # Variables d'environnement
├── .mcp.json            # Config MCP Supabase
├── tailwind.config.js
├── vite.config.ts
└── README.md
```

## Configuration Supabase (optionnel)

Si tu choisis d'utiliser Supabase, tu auras besoin de :

1. **Créer un projet** sur [supabase.com](https://supabase.com)
2. **Récupérer tes clés** dans Settings > API :
   - `VITE_SUPABASE_URL` : URL du projet
   - `VITE_SUPABASE_ANON_KEY` : Clé anon/public
   - `service_role key` : Pour le MCP (optionnel)

## Commandes disponibles

Une fois ton projet créé :

```bash
npm run dev      # Démarre le serveur de dev
npm run build    # Build pour la production
npm run preview  # Preview du build
```

## Support

Des questions ? Ouvre une issue sur ce repo.
