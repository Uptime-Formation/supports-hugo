---
title: "3 - TP Configuration et Premier Pas"
weight: 1030
---

## _Mise en route de l'environnement_

> ⏱ **45 min**

> **Outils :** OpenCode (TUI), Roo Code (VSCode) ou Codex CLI

L'idée est d'aussi pouvoir tester différents modèles et Codex CLI ne le permet pas.

<!-- 
---

# L'écosystème en 3 minutes

Trois catégories d'outils, un seul principe : **apportez votre propre clé API**.

| Catégorie | Exemples | Usage |
|-----------|----------|-------|
| **Agents TUI** (terminal) | Codex, OpenCode, Claude Code | Session autonome dans le projet |
| **Assistants IDE** | Cursor, Copilot, Cline | Autocomplete + chat dans l'éditeur |
| **Bots PR** | Jules, CodeRabbit | Revue automatique sur les Pull Requests |

Cette formation se concentre sur les agents TUI — les plus puissants pour coder et les plus transparents sur ce qu'ils font.

**Pourquoi une clé OpenRouter plutôt qu'un abonnement ?** Accès à 200+ modèles sans vendor lock-in : frugal (Gemini Flash) comme premium (Claude Sonnet). Vous changez de modèle sans changer d'outil. -->

---

# Prérequis

- [ ] Git configuré
- [ ] Un éditeur de code (VSCode recommandé)
- [ ] La clé OpenRouter fournie par le formateur

---

# Étape 1 : Configurer votre clé OpenRouter

Vous avez reçu une clé OpenRouter (`sk-or-v1-...`). Exportez-la :

```bash
export OPENROUTER_API_KEY="sk-or-v1-..."
# Ajouter à ~/.bashrc ou ~/.zshrc pour la rendre persistante
```

**Pourquoi OpenRouter ?**
- Accès à 200+ modèles via une seule clé
- Frugal (Gemini Flash ~$0.10/1M) comme premium (Claude Opus ~$15/1M)
- Pas de vendor lock-in

---

# Étape 2 : Installation

- **OpenCode :** <https://opencode.ai/>

- **Roo Code :** extension VSCode — chercher "Roo Code" dans le marketplace VSCode

- **Codex CLI :**

```bash
npm install -g @openai/codex
codex --version
```

---

# Étape 3 : Configuration

**Roo Code :** dans les paramètres de l'extension VSCode, renseigner l'URL `https://openrouter.ai/api/v1` et la clé OpenRouter.

**OpenCode — option interactive :**

```
/connect   → recherchez "OpenRouter" → saisissez la clé fournie par le formateur
/models    → sélectionnez le modèle souhaité
```

De nombreux modèles OpenRouter sont préchargés ; `/models` vous laisse en changer à tout moment.


**Codex CLI** se configure via variables d'environnement :

```bash
export OPENAI_API_KEY="${OPENROUTER_API_KEY}"
export OPENAI_BASE_URL="https://openrouter.ai/api/v1"
```
<!-- export OPENAI_MODEL=""  -->


---

# Choisir son app de travail

**[Microblog](https://github.com/miguelgrinberg/microblog)** est l'app démo de cette formation : un Twitter en Python créé chapitre par chapitre dans le [Flask Mega-Tutorial](https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-i-hello-world). Chaque branche correspond à un chapitre.



<!-- En plus compliqué à déployer, **[Comparia](https://github.com/betagouv/comparia)** est un site de comparaison de modèles d'IA développé par beta.gouv.fr. -->

Vous pouvez aussi appliquer les mêmes exercices à votre propre projet.
<!-- git clone https://github.com/betagouv/comparia -->

---

> **Tip — lancer une commande shell directement :** dans OpenCode et Claude Code, préfixez n'importe quelle ligne par `!` pour l'exécuter dans le shell sans créer de message.
> ```
> !git status
> !ls -la
> !make test
> ```
> Utile pour vérifier l'état du projet sans quitter l'agent.

---

# Étape 4 : Cloner et faire marcher Microblog

```bash
git clone https://github.com/miguelgrinberg/microblog
cd microblog
```

**Lancer l'app avec l'aide de l'agent :**

```bash
opencode
```

```
> Fais tourner ce projet en local
```

**Vérification :** l'interface est accessible dans le navigateur.

> **Bonus :** [Comparia](https://github.com/betagouv/comparia) (outil de comparaison de LLMs de beta.gouv.fr) est une vraie app en production si vous voulez un terrain plus complexe.

---

> **opencode.school :** [Lesson 3 — Configuration](https://opencode.school/lessons/configuration/) — va plus loin sur les options du fichier de config global.

# Ressources

- [Microblog — miguelgrinberg](https://github.com/miguelgrinberg/microblog)
- [Flask Mega-Tutorial](https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-i-hello-world)
- [OpenRouter](https://openrouter.ai/)
- [OpenCode GitHub](https://github.com/opencode-ai/opencode)
- [Roo Code VSCode Marketplace](https://marketplace.visualstudio.com/items?itemName=RooVeterinaryInc.roo-cline)
- [Codex CLI GitHub](https://github.com/openai/codex)
