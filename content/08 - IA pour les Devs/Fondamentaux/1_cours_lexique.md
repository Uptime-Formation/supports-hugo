---
title: "1 - Lexique"
weight: 1010
---

## _Les termes du cours_

---

# Token

L'unité de base qu'un LLM manipule. Un token ≈ 4 caractères en anglais, un peu moins en français. "Formation" compte pour 2–3 tokens selon le modèle.

Les providers facturent séparément les tokens **envoyés** (input) et les tokens **générés** (output).

Pour visualiser concrètement comment les tokens sont prédits — probabilités, sélection, température : [Vittascience — simulateur LLM](https://fr.vittascience.com/ia/)

---

# Context window

La quantité maximale de tokens qu'un modèle peut traiter en une seule fois : prompt système, historique de la conversation, fichiers lus, résultats de commandes. Au-delà, le modèle ne voit plus le début.

| Modèle | Context window |
|--------|----------------|
| Claude Sonnet 4 | 200 000 tokens |
| GPT-4o | 128 000 tokens |
| Gemini 2.0 Flash | 1 000 000 tokens |

---

# Tokens d'entrée / de sortie

**Input tokens** — tout ce que le modèle lit avant de répondre : prompt, historique, contenu des fichiers, résultats d'outils. Comptés à chaque échange — le contexte s'accumule.

**Output tokens** — tout ce que le modèle génère : texte, code, appels d'outils. Coûtent 3–5× plus cher que les tokens d'entrée.

---

# Tool call

Mécanisme par lequel un LLM émet une requête structurée (JSON) pour déclencher une action : lire un fichier, lancer une commande, faire une recherche web. L'outil lui renvoie le résultat sous forme de texte, qui réintègre le contexte.

Le LLM ne touche jamais directement l'environnement — c'est l'agent CLI (OpenCode, Codex, Claude Code) qui exécute et renvoie les résultats.

---

# MCP — Model Context Protocol

Standard ouvert (Anthropic, 2024) qui définit comment brancher des outils externes sur un agent : base de données, navigation web, accès à des APIs tierces… Un MCP expose un ensemble d'outils typés que l'agent peut appeler comme n'importe quel tool call natif.

---

# LSP — Language Server Protocol

Protocole de communication entre un éditeur et un "serveur de langage" qui comprend votre code : types inférés, définitions, références, erreurs. Votre IDE l'utilise déjà pour les autocomplétions et le "go to definition". Branché sur un agent, il lui donne une navigation précise du codebase — pas du grep approximatif.

---

# Hallucination (confabulation)

Quand un LLM génère du texte plausible mais incorrect : une API inexistante, un paramètre inventé, un fait inexact. Le modèle ne "sait" pas qu'il hallucine — il prédit le token le plus probable, pas le plus juste.

Réduire l'hallucination : ancrer le modèle dans des faits réels (documentation à jour, fichiers du projet, résultats de recherche web).

---

# System prompt

Texte envoyé au modèle avant la conversation, invisible pour l'utilisateur final, qui définit le rôle, les contraintes et les outils disponibles. Dans les agents TUI, `AGENTS.md` y est souvent inclus automatiquement.

---

# Prompt caching

Optimisation proposée par certains providers (Anthropic, OpenAI) : les tokens d'entrée répétitifs — même `AGENTS.md`, mêmes fichiers de contexte — sont mis en cache. À la session suivante avec le même contexte, le cache est réutilisé à ~10% du prix normal.

TTL chez Anthropic : 5 minutes.

---

# Worktree

Fonctionnalité git permettant d'avoir plusieurs copies de travail d'un même repo simultanément, chacune sur une branche différente. Utile pour faire tourner plusieurs agents en parallèle sur des tâches isolées.

```bash
git worktree add ../feature-auth feature/auth
git worktree add ../feature-export feature/export
# Deux agents, deux branches, sans se marcher dessus
```

> **opencode.school :** [Glossaire](https://opencode.school/glossary) — définitions de référence pour tous les termes de ce cours.
