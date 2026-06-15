---
title: "2 - Typologie des Outils IA"
weight: 1020
---

## _Comprendre l'écosystème sans se perdre_

---

# Apportez votre propre clé API

**Les outils sont largement interchangeables.**
**Apportez votre propre clé API** (via OpenRouter, OpenAI, Anthropic...) et utilisez n'importe quel outil.


# Le flux complet

Ce qui se passe réellement quand vous tapez un prompt :

```
┌─────────────────┐        ┌───────────────────────┐        ┌─────────────────────┐
│  Vous           │──────▶ │  Agent CLI            │──────▶ │  API Provider       │
│  (terminal /    │        │  (OpenCode, Codex,    │        │  (OpenRouter,       │
│   éditeur)      │◀────── │   Claude Code…)       │◀────── │   Anthropic, OpenAI)│
└─────────────────┘        └───────────────────────┘        └─────────────────────┘
        │                           │                                   │
   Vous écrivez              Exécute les                        Le modèle LLM
   un prompt                 tool calls                         génère les tokens
                             localement
```

L'agent CLI est le seul à toucher votre machine. Le provider ne voit que du texte.

---

# Typologie par catégorie

Ne choisissez pas un outil individuel - comprenez les **catégories**.

## Type 1 : Agents TUI (Terminal User Interface)

**Agents quasi-autonomes dans le terminal.**

| Outil | Caractéristiques |
|-------|------------------|
| **OpenCode** | Open source, agnostique, tool calling transparent |
| **Claude Code** | Vendor lock-in Anthropic mais très performant |
| **Codex CLI** | OpenAI, récent |
| **Gemini CLI** | Google, récent |

**Pourquoi OpenCode ?**
- Tool calling **transparent** : vous voyez chaque action
- **Open source** : vous contrôlez
- **Agnostique** : pas lié à un provider

<!-- **Usage :** `opencode run "add authentication to the API"` puis laissez mouliner. -->

---

## Type 2 : Assistants IDE (VSCode et apparentés)

**Autocomplete intelligent + agents légers.**
- **Cursor** 
- **GitHub Copilot**
- **Kilo Code**
- **Cline**
- **Roo Code**: Fork de Cline

---

## Type 3 : Bots PR (CI/CD)

**Agents qui travaillent sur vos Pull Requests.**

| Outil | Caractéristiques |
|-------|------------------|
| **Jules** | Google, travaille en background |
| **GitHub Copilot for PR** | Revue automatique |
| **Manuellement** | ex: Avec Github Actions |
| **Opencode avec Gitlab** | <https://opencode.ai/docs/gitlab/> |

**Usage :** Créez une PR, le bot commente et propose des fixes.

## Type 4 : Antigravity, Opencode Desktop

Des worktrees Git natifs.


---

Choisir son outil

**Question clé : quel est votre workflow ?**

- **Terminal-first, vous aimez contrôler** → OpenCode / Claude Code
- **VSCode habituel, autocomplete suffisant** → Cursor / Copilot
- **Équipe établie, CI/CD mature** → Bots PR

## Openrouter

```bash
# Configuration OpenRouter dans OpenCode
export OPENROUTER_API_KEY="sk-or-v1-..."
# Accès à 200+ modèles
# Gemini Flash (frugal), Claude (qualité), Qwen (multimodal)...
```

---

# Coûts et modèles frugaux

**La facture arrive vite : $500-2000/mois** pour un usage intensif.

GLM 5.1, Gemini Flash, Claude Haiku, Qwen...

Commencez frugal, passez premium pour les décisions critiques.
**Pour le cours :** il est intéressant d'opérer avec des modèles suboptimaux pour observer les comportements erratiques principaux des agents, causés par les modèles LLM qui sont derrière.


## Tour rapide des modèles du moment

---

# Configuration initiale

**Objectif :**
1. Configurer Opencode ou Roo avec votre clé

> **opencode.school :** [Lesson 1 — Installation](https://opencode.school/lessons/installation/) — chemin alternatif pas-à-pas si vous préférez la version guidée interactive.
