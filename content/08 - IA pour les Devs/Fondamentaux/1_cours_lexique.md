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
# Tokens spéciaux / de contrôle

Pour comprendre les **tokens spéciaux** (souvent appelés *special tokens* ou *control tokens*), il faut les imaginer comme la **ponctuation et la grammaire invisibles** que les développeurs utilisent pour parler à l'intelligence artificielle.

Alors que nous utilisons des mots pour communiquer, le modèle utilise ces tokens pour structurer sa "pensée" et savoir quand s'arrêter. Voici un exemple concret de la manière dont ces tokens orchestrent une interaction.

---

### Exemple : Demande de calcul mathématique

Imaginons un utilisateur demandant : *"Combien font 15 multiplié par 4 ?"*

Voici à quoi ressemble la séquence de tokens que le modèle traite en coulisses :

`<system_start>`
Tu es un assistant expert en mathématiques. Utilise une étape de réflexion avant de répondre.
`<system_end>`
`<user_start>`
Combien font 15 multiplié par 4 ?
`<user_end>`
`<thinking>`
L'utilisateur veut multiplier 15 par 4.
Je peux décomposer cela : (10 * 4) + (5 * 4) = 40 + 20 = 60.
La réponse est 60.
`<thinking_end>`
Le résultat de 15 multiplié par 4 est 60.
`<eos>`

### Principaux tokens de contrôle

| Token | Rôle explicatif |
| --- | --- |
| **`<system_start/end>`** | **Le cadre :** Ils délimitent les instructions de comportement données par les développeurs. Cela empêche le modèle de confondre les ordres du créateur avec les questions de l'utilisateur. |
| **`<user_start/end>`** | **L'entrée :** Ils isolent la requête de l'utilisateur. Cela permet au modèle de savoir précisément quelle information il doit traiter. |
| **`<thinking>`** | **Le brouillon :** Ce token signale au modèle qu'il doit générer une réflexion interne (raisonnement logique) avant de produire une réponse finale. Ce texte n'est pas forcément montré immédiatement à l'utilisateur. |
| **`<thinking_end>`** | **La fin de la réflexion :** Il indique que la phase de calcul ou de réflexion est terminée et que le modèle doit maintenant rédiger la réponse. |
| **`<eos>`** | **End Of Sequence (Fin de séquence) :** C'est le plus important. Il indique au modèle que sa génération est complète. Sans ce token, le modèle pourrait continuer à divaguer indéfiniment. |



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
