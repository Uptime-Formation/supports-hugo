---
title: "7 - TP Contexte et coûts"
weight: 1070
---

## Visualiser sa consommation avec codeburn

[codeburn](https://github.com/AgentSeal/codeburn) est un dashboard TUI qui lit les fichiers de session de vos outils IA directement sur disque — sans wrapper, sans clé API — et vous montre où partent vos tokens : par projet, par modèle, par type d'activité.

## Installation

```bash
npm install -g codeburn
# Ou sans installer :
npx codeburn
```

Prérequis : Node.js 22+.

## Exercice

**1. Lancer le dashboard sur vos 7 derniers jours :**

```bash
codeburn
```

Naviguez avec les flèches pour changer la période. Tapez `c` pour comparer les modèles, `o` pour les suggestions d'optimisation.

**2. Résumé rapide en une ligne :**

```bash
codeburn status
```

**3. Identifier les commandes qui brûlent le plus de tokens :**

```bash
codeburn optimize
```

Repérez les tool calls inutilement coûteux (lecture de fichiers entiers, commandes verbose…).

**4. Export pour archiver :**

```bash
codeburn report -p 30days
codeburn export
```

**Ce que vous cherchez :** quel outil / quel projet consomme le plus ? Y a-t-il des pics inexpliqués ? Les suggestions `optimize` correspondent-elles à ce que vous observez ?

---

# Étape 10 : Réduire sa consommation avec rtk

[rtk](https://github.com/rtk-ai/rtk) est un proxy CLI qui filtre et compresse les sorties de commandes avant qu'elles n'entrent dans le contexte de votre agent — 60 à 90% de tokens en moins sur les commandes courantes (`git status`, `ls`, `diff`, `pytest`…).

## Installation

```bash
# Linux/macOS :
curl -fsSL https://raw.githubusercontent.com/rtk-ai/rtk/refs/heads/master/install.sh | sh

# macOS via Homebrew :
brew install rtk

# Vérifier :
rtk --version
```

## Initialisation

```bash
rtk init -g --opencode      # OpenCode plugin
rtk init --agent cline          # Cline / Roo Code
rtk init -g --codex             # Codex (OpenAI)
rtk init -g                     # Claude Code / Copilot
rtk init -g --gemini            # Gemini CLI
```

Redémarrez votre outil IA après l'init — les commandes se réécrivent automatiquement via un hook Bash.

## Exercice

**1. Comparer la sortie brute vs. filtrée :**

Dans un repo avec de l'activité Git :

```bash
# Sortie brute :
git status

# Sortie filtrée pour l'agent :
rtk git status
```

Observez la différence de volume. Faites pareil avec `rtk git log` et `rtk git diff`.

**2. Lancer une session avec rtk actif :**

Lancez votre agent sur une tâche courante (ajout d'une feature, correction d'un bug). Comparez le coût affiché avec une session équivalente sans rtk.

**3. Voir les économies accumulées :**

```bash
rtk gain
```

**4. Découvrir les opportunités d'optimisation :**

```bash
rtk discover
```

**Commandes disponibles :**

| Catégorie | Exemples |
|-----------|---------|
| Fichiers | `rtk ls`, `rtk read`, `rtk find`, `rtk grep`, `rtk diff` |
| Git | `rtk git status/log/diff/push/pull` |
| Tests | `rtk pytest`, `rtk jest`, `rtk cargo test` |
| Build/Lint | `rtk tsc`, `rtk ruff check`, `rtk cargo build` |
| Containers | `rtk docker ps`, `rtk kubectl pods` |

**Ce que vous cherchez :** sur quel type de commande le gain est-il le plus fort ? Est-ce que la sortie filtrée reste suffisante pour que l'agent travaille correctement ?

---

---

# Compression de contexte automatique avec opencode-dcp

[opencode-dcp](https://github.com/tarquinen/opencode-dcp) est un plugin OpenCode qui compresse automatiquement le contexte de la conversation quand il grossit, sans intervention manuelle. Utile pour les longues sessions où le contexte s'accumule et commence à coûter cher.

```json
"plugin": ["@tarquinen/opencode-dcp@latest"]
```

Une fois installé, la compression se déclenche en arrière-plan — vous n'avez pas à penser à lancer `/compact` manuellement.

---

# Livrable

À la fin de ce TP :

- [ ] Mesuré sa consommation sur une tâche réelle
- [ ] Identifié deux tâches : une "Plan" (raisonnement), une "Act" (exécution)
- [ ] Configuré un profil frugal dans son outil
- [ ] Essayé le pattern pingre (Gemini gratuit → modèle cheap)
- [ ] Visualisé sa consommation avec codeburn (`codeburn optimize`)
- [ ] Réduit les tokens d'une session avec rtk (`rtk gain`)

---

# Checkpoint

**Question clé :** Pour quelle tâche d'aujourd'hui auriez-vous pu utiliser un modèle moins cher ?

**Pattern retenu :** Phase Plan = raisonnement fort. Phase Act = modèle frugal.

---
