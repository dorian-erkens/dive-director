# Dive Director — Assistant DP pour Claude Code

> **[English below](#english)**

[![Claude Code](https://img.shields.io/badge/Built%20with-Claude%20Code-blueviolet)](https://claude.com/claude-code)
[![FFESSM](https://img.shields.io/badge/Norme-FFESSM%20%2F%20Code%20du%20Sport-blue)](https://ffessm.fr)
[![SHOM MCP](https://img.shields.io/badge/SHOM%20MCP-4796%2B%20%C3%A9paves-teal)](https://github.com/dorian-erkens/mcp-shom-wrecks)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

Assistant intelligent pour **Directeur de Plongée (DP)** basé sur [Claude Code](https://docs.anthropic.com/en/docs/claude-code). Conçu pour le club [Caen Ouistreham Plongée (COP)](https://caen-ouistreham-plongee.org/), adaptable à tout club de plongée français.

---

## Ce que ça fait

Planification complète d'une sortie plongée en une commande :

```
/plan-dive
> On plonge samedi sur le Svenner, 3 N2, 2 N3 et 1 GP.
> Vent SW 12 noeuds, Douglas 2.
```

L'assistant orchestre **7 agents spécialisés** qui suivent le workflow officiel **DPE-PN5** (Directeur de Plongée en Exploration, FFESSM) :

| Phase | Objectif | Agents |
|:---:|---|---|
| 1 | **Cadrage** — type d'Explo (1/2/3), marées et profondeurs réelles, plongeurs inscrits | tide-calculator |
| 2 | **Faisabilité** — météo go/no-go, puis choix du site adapté au cadrage | dive-conditions, wreck-finder |
| 3 | **Organiser** — aptitudes, palanquées, fiche de sécurité, transit | ffessm-diving-expert, safety-sheet, boat-specs, nautical-position-calculator |
| 4 | **Sécuriser** — briefing complet avec toutes les infos pour le DP | (synthèse) |

Le type d'Explo est décidé **3 mois à l'avance** (Explo 1 ≤ 20m, Explo 2 ≤ 40m, Explo 3 ≤ 60m). La météo peut **annuler** la sortie mais ne downgrade jamais le type d'Explo — en revanche elle influence le choix du site.

Si une étape est bloquante (météo feu rouge, profondeur hors prérogatives, pas assez de GP...), l'assistant **stoppe immédiatement** et explique pourquoi.

## Les 7 agents

| Agent | Rôle | Données |
|---|---|---|
| `wreck-finder` | Base d'épaves de la Baie de Seine | [SHOM MCP](https://github.com/dorian-erkens/mcp-shom-wrecks) (4 796+ épaves) + données club COP |
| `tide-calculator` | Marées, coefficients, fenêtres d'étale | [maree.info](https://maree.info/25) |
| `dive-conditions` | Évaluation météo-marine go/no-go | Matrice Douglas + vent sectoriel |
| `ffessm-diving-expert` | Réglementation FFESSM / Code du Sport | Niveaux, prérogatives, profondeurs |
| `safety-sheet` | Fiche de sécurité + palanquées | Article A.322-72 |
| `boat-specs` | Caractéristiques du bateau | CIPI'ONE (8,80m, 260cv, 19 pers.) |
| `nautical-position-calculator` | Calculs GPS et navigation | Loxodromie, estime |

## Prérequis

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installé
- Un abonnement Anthropic (les agents utilisent le modèle `sonnet` par défaut)
- Optionnel : [mcp-shom-wrecks](https://github.com/dorian-erkens/mcp-shom-wrecks) pour les données d'épaves SHOM (4 796+ épaves en temps réel)

## Installation

```bash
# Cloner le repo
git clone https://github.com/dorian-erkens/dive-director.git
cd dive-director

# Lancer Claude Code dans le répertoire
claude
```

Claude Code détecte automatiquement le `CLAUDE.md` et les agents dans `.claude/agents/`.

### Configuration du SHOM MCP (optionnel mais recommandé)

Le [SHOM MCP](https://github.com/dorian-erkens/mcp-shom-wrecks) donne accès à **4 796+ épaves** de la base du Service Hydrographique et Océanographique de la Marine.

```bash
# Installer le MCP
npm install -g mcp-shom-wrecks
# Ou cloner et builder
git clone https://github.com/dorian-erkens/mcp-shom-wrecks.git
cd mcp-shom-wrecks && npm install && npm run build
```

Ajouter dans `~/.claude/settings.json` :

```json
{
  "mcpServers": {
    "shom-wrecks": {
      "command": "node",
      "args": ["/chemin/vers/mcp-shom-wrecks/dist/index.js"]
    }
  }
}
```

## Utilisation

### Commande principale

```
/plan-dive
```

Fournir : site (ou laisser le choix), date, liste de plongeurs avec niveaux, conditions météo.

### Requêtes simples

```
"C'est quoi le Svenner ?"           -> wreck-finder
"Marées demain ?"                    -> tide-calculator
"Vent NW 18 nds, on sort ?"         -> dive-conditions
"Un N2 peut aller à 35m ?"          -> ffessm-diving-expert
"Organise ces palanquées"           -> safety-sheet
"Combien de temps pour le M39 ?"    -> boat-specs
"Cap vers le Northgate ?"           -> nautical-position-calculator
```

## Structure du projet

```
dive-director/
├── CLAUDE.md                              # Orchestrateur principal
├── README.md
├── .claude/
│   ├── agents/                            # 7 agents spécialisés
│   │   ├── wreck-finder.md
│   │   ├── tide-calculator.md
│   │   ├── dive-conditions.md
│   │   ├── ffessm-diving-expert.md
│   │   ├── safety-sheet.md
│   │   ├── boat-specs.md
│   │   └── nautical-position-calculator.md
│   ├── commands/
│   │   └── plan-dive.md                   # Commande /plan-dive
│   └── agent-memory/                      # Mémoire persistante par agent
│       ├── wreck-finder/
│       ├── tide-calculator/
│       ├── dive-conditions/
│       ├── boat-specs/
│       └── nautical-position-calculator/
```

## Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    UTILISATEUR (Directeur de Plongée)               │
│                                                                     │
│   CLI : Claude Code                Web UI (WIP)                     │
│   ┌──────────────┐                 ┌──────────────────────────┐     │
│   │  /plan-dive  │                 │  React + Tailwind +      │     │
│   │  ou question │                 │  Leaflet (carte épaves)  │     │
│   │  libre       │                 │  Chat / Inspector / Map  │     │
│   └──────┬───────┘                 └────────────┬─────────────┘     │
└──────────┼──────────────────────────────────────┼───────────────────┘
           │                                      │
           ▼                                      ▼
┌─────────────────────┐              ┌──────────────────────────┐
│   CLAUDE CODE       │              │  BACKEND — FastAPI       │
│   (Orchestrateur)   │              │                          │
│                     │              │  routers/                │
│  CLAUDE.md          │              │  ├── chat.py  → Claude   │
│  = routage +        │              │  └── wrecks.py → SHOM   │
│  workflow DPE-PN5   │              │  services/               │
│                     │              │  ├── claude.py (SDK)     │
└──────────┬──────────┘              │  └── shom.py  (WFS)     │
           │                         └──────────────────────────┘
           ▼
┌──────────────────────────────────────────────────────────────┐
│                    7 AGENTS SPÉCIALISÉS                       │
│                    (.claude/agents/*.md)                      │
│                                                              │
│  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐ │
│  │ wreck-finder   │  │ tide-calculator│  │dive-conditions │ │
│  │ (épaves)       │  │ (marées)       │  │(météo go/nogo) │ │
│  └───────┬────────┘  └───────┬────────┘  └────────────────┘ │
│  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐ │
│  │ ffessm-expert  │  │ safety-sheet   │  │ boat-specs     │ │
│  │(réglementation)│  │ (fiche sécu)   │  │ (CIPI'ONE)     │ │
│  └────────────────┘  └────────────────┘  └────────────────┘ │
│  ┌────────────────┐                                         │
│  │ nautical-pos   │  agent-memory/ (mémoire persistante)    │
│  │ (navigation)   │                                         │
│  └───────┬────────┘                                         │
└──────────┼───────────────────────────────────────────────────┘
           │
           ▼
┌──────────────────────────────────────────────────────────────┐
│                    SOURCES DE DONNÉES                         │
│                                                              │
│  ┌─────────────────────────┐  ┌────────────────────────────┐│
│  │  MCP SHOM WRECKS        │  │  maree.info/25             ││
│  │  (Node.js / TypeScript)  │  │  (WebFetch — Ouistreham)  ││
│  │  4 outils MCP            │  └────────────────────────────┘│
│  │       │                  │                                │
│  │       ▼                  │  ┌────────────────────────────┐│
│  │  API WFS SHOM (HTTPS)    │  │  Données embarquées       ││
│  │  4 796+ épaves           │  │  (dans les agents .md)    ││
│  └─────────────────────────┘  │  • Specs CIPI'ONE          ││
│                               │  • Grilles FFESSM          ││
│                               │  • Tables Douglas          ││
│                               │  • Épaves COP favorites    ││
│                               └────────────────────────────┘│
└──────────────────────────────────────────────────────────────┘
```

### Workflow `/plan-dive`

```
          DP lance /plan-dive
                 │
      ┌──────────▼──────────┐
      │  Phase 1 — CADRAGE  │  tide-calculator → maree.info
      │  Type Explo + marée │  Profondeurs réelles, étale
      │  + plongeurs        │
      └──────────┬──────────┘
                 │
      ┌──────────▼──────────┐
      │ Phase 2 — FAISAB.   │  dive-conditions → go / 🛑 STOP
      │  Météo, règle 6 NM  │  nautical-pos → distance abri
      │  Choix du site       │  wreck-finder → MCP SHOM
      └──────────┬──────────┘
                 │ DP choisit
      ┌──────────▼──────────┐
      │ Phase 3 — ORGANISER │  ffessm-expert → aptitudes
      │  Palanquées, sécu   │  safety-sheet → fiche A.322-72
      │  Transit             │  boat-specs + nautical-pos
      └──────────┬──────────┘
                 │
      ┌──────────▼──────────┐
      │ Phase 4 — BRIEFING  │  Synthèse complète
      │  Nav + Sécu + Site  │  → Décision du DP
      └─────────────────────┘
```

### Protocoles de communication

| Lien | Protocole |
|---|---|
| Claude Code ↔ Agents | Subagent spawning natif (`.claude/agents/*.md`) |
| Claude Code ↔ MCP SHOM | Model Context Protocol (stdio, JSON-RPC 2.0) |
| Claude Code ↔ maree.info | WebFetch (HTTP GET + parsing HTML) |
| Web App ↔ Backend | REST API (FastAPI — `/chat`, `/wrecks`) |
| Backend ↔ Claude | Anthropic Python SDK (`anthropic`) |
| Backend ↔ SHOM | HTTP direct vers API WFS SHOM |

### Repos liés

| Repo | Tech | Rôle |
|---|---|---|
| [**dive-director**](https://github.com/dorian-erkens/dive-director) | Markdown + Claude Code | Orchestrateur, 7 agents, `/plan-dive`, mémoire |
| [**mcp-shom-wrecks**](https://github.com/dorian-erkens/mcp-shom-wrecks) | TypeScript + Node.js + MCP SDK | Serveur MCP — 4 796+ épaves SHOM |
| **dive-director-app** *(WIP)* | React 19 / Vite / Tailwind / Leaflet + FastAPI | Interface web : chat, carte, inspecteur |

## Adaptation à votre club

Ce système est conçu pour le COP mais facilement adaptable :

1. **`boat-specs.md`** — Remplacer les caractéristiques du CIPI'ONE par votre bateau
2. **`wreck-finder.md`** — Adapter la base d'épaves à votre zone de plongée
3. **`tide-calculator.md`** — Changer le port de référence (code maree.info)
4. **`dive-conditions.md`** — Ajuster les seuils de vent selon votre exposition côtière
5. **`nautical-position-calculator.md`** — Modifier le point de départ

Les agents `ffessm-diving-expert` et `safety-sheet` sont génériques (réglementation française) et fonctionnent pour tout club FFESSM.

## Réglementation

Les informations réglementaires sont basées sur :
- **Code du Sport** — Articles A322-71 à A322-115
- **Manuel de Formation Technique FFESSM** (2024-2025)
- **Cursus DPE-PN5** — Formation Directeur de Plongée en Exploration

**Avertissement** : cet outil est une aide à la décision. La responsabilité du Directeur de Plongée reste entière. Les données de marée, météo et positions GPS doivent toujours être vérifiées avec les sources officielles avant toute sortie.

## Licence

MIT

## Contributeurs

Construit avec [Claude Code](https://claude.com/claude-code) (Anthropic) pour le club [Caen Ouistreham Plongée](https://caen-ouistreham-plongee.org/).

---

<a id="english"></a>

## English

**Dive Director** is an AI-powered assistant for scuba **Dive Directors** (Directeur de Plongée), built on [Claude Code](https://docs.anthropic.com/en/docs/claude-code). It orchestrates 7 specialized agents to plan complete dive trips following the official French diving federation (FFESSM) DPE-PN5 workflow:

- **Wreck identification** from the SHOM database (4,796+ wrecks) via [mcp-shom-wrecks](https://github.com/dorian-erkens/mcp-shom-wrecks)
- **Tide calculation** for real depth at dive time
- **Weather assessment** with go/no-go matrix
- **Regulation check** (FFESSM certification levels vs. depth)
- **Team organization** (buddy groups, safety sheet)
- **Boat logistics** (transit time, fuel, capacity)
- **Navigation** (GPS waypoints, course, distance)

Designed for [Caen Ouistreham Plongée (COP)](https://caen-ouistreham-plongee.org/) diving from the port of Ouistreham (Normandy, France), but adaptable to any French dive club. See the [Adaptation section](#adaptation-à-votre-club) and [Architecture diagram](#architecture) for details.
