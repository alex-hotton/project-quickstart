# Project Quickstart

Crée un projet web moderne en quelques minutes avec Claude Code. Setup complet, testé, prêt à développer.

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

## Ce que fait `/initproject`

```
PHASE 1 → Questions (nom du projet, besoin backend ?)
    ↓
PHASE 2 → Setup technique (Vite, React, Tailwind, i18n, etc.)
    ↓
PHASE 3 → Config Supabase (si demandé)
    ↓
PHASE 4 → Test de validation (mini-app qui affiche une table Supabase)
    ↓
PHASE 5 → Demande ton PRD et nettoie la table de test
    ↓
PHASE 6 → Génère CLAUDE.md (conventions génériques)
    ↓
PHASE 7 → Crée le repo GitHub et push
```

## Prérequis

- [Node.js](https://nodejs.org/) 18+
- [Claude Code](https://claude.com/code) installé
- [GitHub CLI](https://cli.github.com/) (pour créer le repo automatiquement)

## Utilisation

### 1. Clone ce repo avec le nom de ton projet

```bash
git clone https://github.com/alex-hotton/project-quickstart.git nom-de-ton-projet
cd nom-de-ton-projet
```

Le dossier cloné sera transformé en ton projet (pas de nouveau dossier créé).

### 2. Lance Claude Code

```bash
claude
```

### 3. Exécute la commande

```
/initproject
```

### 4. Suis les étapes

Claude Code va :

1. **Te poser des questions** : nom du projet, besoin d'un backend ?
2. **Setup le projet** : installe tout, configure Tailwind, shadcn, etc.
3. **Configurer Supabase** : te guide pour créer le projet et récupérer les clés
4. **Tester la connexion** : crée une mini-app qui affiche une table de test
5. **Demander ton PRD** : pour comprendre ce que tu veux construire
6. **Générer CLAUDE.md** : le contexte complet du projet pour Claude Code
7. **Créer le repo GitHub** : et push le premier commit

## Résultat final

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
├── CLAUDE.md             # ⭐ Contexte du projet basé sur ton PRD
├── .env                  # Variables d'environnement
├── .mcp.json            # Config MCP Supabase
├── tailwind.config.js
├── vite.config.ts
└── README.md
```

## Le fichier CLAUDE.md

C'est le fichier le plus important. Il contient :

- Description du projet
- Stack technique
- Structure des dossiers
- Fonctionnalités (MVP et futures)
- Modèle de données
- Routes/pages
- Conventions de code

Claude Code le lit automatiquement et comprend ton projet.

## Configuration Supabase

Si tu choisis d'utiliser Supabase, tu auras besoin de :

1. **Créer un projet** sur [supabase.com](https://supabase.com)
2. **Récupérer tes clés** dans Settings > API :
   - `VITE_SUPABASE_URL` : URL du projet
   - `VITE_SUPABASE_ANON_KEY` : Clé anon/public
   - `service_role key` : Pour le MCP

## Commandes disponibles

Une fois ton projet créé :

```bash
npm run dev      # Démarre le serveur de dev
npm run build    # Build pour la production
npm run preview  # Preview du build
```

## Support

Des questions ? Ouvre une issue sur ce repo.
