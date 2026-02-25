# Dive Director — Assistant DP pour Claude Code

Assistant intelligent pour **Directeur de Plongée (DP)** basé sur [Claude Code](https://docs.anthropic.com/en/docs/claude-code). Conçu pour le club [Caen Ouistreham Plongée (COP)](https://caen-ouistreham-plongee.org/), adaptable à tout club de plongée français.

## Ce que ça fait

Planification complète d'une sortie plongée en une commande :

```
/plan-dive
> On plonge samedi sur le Svenner, 3 N2, 2 N3 et 1 GP.
> Vent SW 12 nœuds, Douglas 2.
```

L'assistant orchestre **7 agents spécialisés** qui suivent le workflow officiel **DPE-PN5** (Directeur de Plongée en Exploration, FFESSM) :

1. **Choisir un site** → identification de l'épave, profondeur réelle selon la marée, conditions météo
2. **Réglementation** → vérification des aptitudes par rapport à la profondeur
3. **Organiser** → composition des palanquées, fiche de sécurité, calcul du transit
4. **Sécuriser** → briefing complet avec toutes les infos pour le DP

## Les 7 agents

| Agent | Rôle | Données |
|---|---|---|
| `wreck-finder` | Base d'épaves (34 épaves cataloguées) | SHOM MCP + données club |
| `tide-calculator` | Marées, coefficients, étale | [maree.info](https://maree.info/25) |
| `dive-conditions` | Évaluation météo go/no-go | Matrice Douglas + vent sectoriel |
| `ffessm-diving-expert` | Réglementation FFESSM / Code du Sport | Niveaux, prérogatives, profondeurs |
| `safety-sheet` | Fiche de sécurité + palanquées | Article A.322-72 |
| `boat-specs` | Caractéristiques du bateau | CIPI'ONE (8,80m, 260cv, 19 pers.) |
| `nautical-position-calculator` | Calculs GPS et navigation | Loxodromie, estime |

## Prérequis

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installé
- Un abonnement Anthropic (les agents utilisent le modèle `sonnet` par défaut)
- Optionnel : [mcp-shom-wrecks](https://github.com/dorian-erkens/mcp-shom-wrecks) pour les données d'épaves SHOM en temps réel

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

Le SHOM MCP donne accès à **4 796+ épaves** de la base du Service Hydrographique et Océanographique de la Marine.

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
"C'est quoi le Svenner ?"           → wreck-finder
"Marées demain ?"                    → tide-calculator
"Vent NW 18 nds, on sort ?"         → dive-conditions
"Un N2 peut aller à 35m ?"          → ffessm-diving-expert
"Organise ces palanquées"           → safety-sheet
"Combien de temps pour le M39 ?"    → boat-specs
"Cap vers le Northgate ?"           → nautical-position-calculator
```

## Structure du projet

```
├── CLAUDE.md                              # Orchestrateur principal
├── .claude/
│   ├── agents/                            # 7 agents spécialisés
│   │   ├── wreck-finder.md
│   │   ├── tide-calculator.md
│   │   ├── dive-conditions.md
│   │   ├── ffessm-diving-expert.md
│   │   ├── safety-sheet.md
│   │   ├── boat-specs.md
│   │   └── nautical-position-calculator.md
│   └── commands/
│       └── plan-dive.md                   # Commande /plan-dive
```

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
