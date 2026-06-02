---
title: "10 - TP Skills & Tests visuels"
weight: 1100
draft: false
---

## _Plan → Build → Review → Retro_

> ⏱ **1h30**

---

# Les 4 piliers

Un projet bien structuré pour l'IA a besoin de :

1. **AGENTS.md** - Contexte permanent pour l'agent
2. **Makefile / run.sh** - Commandes reproductibles
3. **Docker** - Environnement isolé et défini
4. **README.md par dossier** - Documentation

---

# Skills & Délégation multi-agents

La plupart des outils proposent d'utiliser plusieurs agents préconfigurés pour certaines tâches, appelés "skill", "mode", "agent" ou "workflow" selon les outils. Dans la pratique, cela revient surtout à spécifier un (pré-)prompt particulier pour qu'un agent classique se concentre sur une problématique particulière. On peut ainsi créer des workflows plus complexes en demandant à un agent de déléguer des sous-tâches à un autre agent. Pour ce TP par exemple : un agent planifie, puis un autre exécute le plan, et un dernier enlève les parties inutiles : le "slop".

Un skill peut se spécialiser de deux façons :

- **Autour d'un outil / MCP** : il précise à l'agent quels outils appeler, dans quel ordre, et comment interpréter les résultats.
- **Autour d'un workflow** : il encode des étapes, des conseils, des commandes à lancer — sans dépendre d'un MCP particulier.

Les deux approches se combinent : un bon skill peut à la fois décrire un workflow structuré ET indiquer les outils MCP à utiliser.

## Créer un skill dans OpenCode

Un skill est un dossier contenant un fichier `SKILL.md`. OpenCode cherche les skills dans :
- **Projet :** `.opencode/skills/<nom>/SKILL.md`
- **Global :** `~/.config/opencode/skills/<nom>/SKILL.md`

**Astuce :** [opencode.school](https://opencode.school/) propose une GUI web pour créer et gérer des skills sans toucher au filesystem.

Pour pouvoir gérer ces agents, on pourra faire référence à ce skill par exemple : [smux/SKILL.md](https://github.com/ShawnPana/smux/blob/main/skills/smux/SKILL.md).

```markdown
---
name: mon-skill
description: Ce que fait ce skill (utilisé par l'agent pour décider de l'activer)
---

## Instructions

Ce que l'agent doit faire quand ce skill est activé.
```

L'agent voit les skills disponibles et les charge à la demande via son outil `skill`. L'invocation est automatique si la tâche correspond à la description du skill.

## Créer un skill dans Codex CLI

Même principe : un dossier avec `SKILL.md`, placé dans `.agents/skills/` à la racine du repo (ou `~/.agents/skills/` pour usage global).

```markdown
---
name: mon-skill
description: Ce que fait ce skill
---

Instructions pour Codex.
```

**Invocation explicite :** tapez `$mon-skill` dans votre prompt.  
**Invocation implicite :** Codex active automatiquement le skill si votre tâche correspond à sa description.

Pour créer un skill interactivement : lancez `$skill-creator`.

---

# Construire le skill `tester-mon-app`

Un cas d'usage concret pour illustrer les skills : formaliser comment tester visuellement une application. Ce skill combine les deux dimensions — workflow structuré et outils MCP — et résout un vrai problème de terrain : comment montrer à l'agent ce qu'on voit à l'écran, sans exploser les coûts en tokens.

## Multimodal : quand et comment

| Cas d'usage | Description |
|-------------|-------------|
| **Screenshot d'erreur** | Montrer une erreur UI plutôt que de la décrire en texte |
| **Mockup → code** | Transformer un design en HTML/CSS |
| **Diagramme d'archi** | Analyser un schéma et suggérer une implémentation |
| **Debug visuel** | "Pourquoi la page s'affiche comme ça ?" |

| Limitation | Impact |
|------------|--------|
| Coût élevé | Une image = 500–2000 tokens selon la résolution |
| Hallucination visuelle | Le modèle peut "lire" du texte qui n'existe pas |
| Résolution limitée | Les détails fins sont souvent manqués |

**L'astuce Playwright :** au lieu d'envoyer une image (lourd), le MCP Playwright retourne une représentation textuelle du DOM. Aucun token d'image, même précision pour naviguer la structure. C'est ce que notre skill va encoder.

## Créer le skill

Créer le fichier `.opencode/skills/tester-mon-app/SKILL.md` (ou `.agents/skills/tester-mon-app/SKILL.md` pour Codex) :

```markdown
---
name: tester-mon-app
description: Tester visuellement l'application localement — snapshot DOM, screenshot d'erreur, ou analyse de mockup. Utiliser ce skill pour tout ce qui implique de "regarder" la page.
---

## Workflow

1. Lancer l'application si elle ne tourne pas déjà (`make dev` ou `npm run dev`)
2. Selon la tâche :
   - **Vérifier la structure d'une page** → utiliser l'outil MCP `browser_snapshot` — retourne du texte, pas une image
   - **Analyser une erreur visuelle** → prendre un screenshot uniquement si le snapshot ne suffit pas
   <!-- - **Transformer un mockup en code** → attacher l'image du mockup et générer le HTML/CSS correspondant -->

## Conseils

- Toujours préférer `browser_snapshot` à `browser_screenshot` : 10 à 20× moins de tokens
<!-- - Pour les SPAs, attendre que le contenu principal soit chargé avant de prendre le snapshot -->
<!-- - Si la page nécessite une authentification, naviguer vers la page de connexion et compléter le flux d'abord -->
- Toujours préciser ce qui est attendu vs ce qui est observé pour aider au diagnostic


## Tester le skill

```
> $tester-mon-app  Vérifie que la page de comparaison de Comparia s'affiche correctement
```

Observez : l'agent charge le skill, lance l'app si nécessaire, utilise `browser_snapshot` et retourne une analyse textuelle — sans aucun token d'image.

---

# Objectif

Réaliser une feature en suivant le cycle complet : plan → build → PR review → retro AGENTS.md.


---

# Choisir la feature

Quelque chose qui touche plusieurs fichiers et nécessite au moins un test :
- Export CSV des comparaisons
- Historique des sessions avec persistance
- Mode "personnage" persistant entre les messages (reprend l'idée du TP 5 côté backend)
- **Une seule conversation** — Comparia affiche actuellement deux conversations en parallèle. Simplifier à une seule réduit la surface d'état côté frontend et backend, sans retirer la comparaison (on peut conserver les deux modèles côte à côte sur un même échange).
- **Historique côté client** — Sauvegarder les comparaisons dans le localStorage pour les retrouver après rechargement.

<!-- Autres idées solides :
- Export de la comparaison en Markdown / JSON (un bouton, un endpoint)
- Partager une comparaison via URL (sérialiser l'état dans les query params)
- Épingler un modèle favori par utilisateur (localStorage)
-->

<!-- Idées plus incertaines :
- Système de notation par réponse (thumbs up/down) — nécessite de la persistance côté serveur
- Bibliothèque de prompts système réutilisables
- Comparaison à 3 modèles (UI non triviale)
-->

---

# Phase 1 — PLAN (pas de code encore)

```
> Je veux implémenter [feature] dans Comparia.
  Analyse le projet et propose un plan :
  - Quels fichiers créer ou modifier ?
  - Quelle est l'architecture proposée ?
  - Quels sont les risques ?
  Ne commence pas à coder. Attends ma validation.
```

Lisez le plan, questionnez les choix. Validez ou demandez des ajustements avant de passer à la suite.

---

# Phase 2 — BUILD

```
> Plan validé. Implémente étape par étape.
  Lance make test après chaque étape.
  Commits atomiques.
```

**Observations à faire pendant l'implémentation :**

- L'agent a-t-il pensé à la **protection misclick** ? (confirmation avant action destructive, bouton désactivé pendant le chargement, double-submit impossible…)
<!-- - Est-il en train de corriger l'environnement et les tests **dans le même commit** ? Si oui, les commits ne sont pas atomiques. -->

---

# Phase 3 — Cleanup

S'assurer que l'agent lance lui-même le skill "Nettoyage de code".

**Codex / OpenCode :**
```
> Passe en revue le code qu'on vient d'écrire.
  Retire les abstractions superflues, les fonctions intermédiaires inutiles.
  Ne change pas le comportement.
```

**Claude Code :**
```
> /simplify
```

---
<!-- 
# Phase 4 — Review

**Codex / OpenCode :**
```
> Passe en revue cette PR comme un dev senior sceptique.
  Identifie les problèmes avant que ça parte en prod.
```

---

# Phase 5 — Retro → AGENTS.md

```
> Résume ce qu'on vient de faire. Qu'est-ce qui t'a manqué comme contexte ?
  Je veux mettre à jour AGENTS.md.
```

L'agent identifie les lacunes de contexte rencontrées. Vous les ajoutez à `AGENTS.md` — la prochaine session repart d'une base plus solide.

**Ce qu'on ajoute typiquement :**
- Contraintes oubliées ("NE PAS modifier les migrations")
- Patterns récurrents du projet ("toujours utiliser le service layer")
- Outils disponibles qu'il ne connaissait pas

---

# Livrable

- [ ] Feature implémentée et testée (`make test` passe)
- [ ] Skill `tester-mon-app` créé et testé sur Comparia
- [ ] PR créée avec review documentée
- [ ] `AGENTS.md` mis à jour après retro

---

# Checkpoint

**Pattern retenu :** Plan d'abord, code ensuite, retro toujours.

**Question clé :** Qu'est-ce que la retro a révélé que votre AGENTS.md ne couvrait pas ?
-->

---

Ressources :

- <https://opencode.ai/docs/skills/>
- <https://developers.openai.com/codex/skills>

> **opencode.school :** [Lesson 8 — Skills](https://opencode.school/lessons/skills/) — installer des skills existants avec `npx skills` et en créer depuis l'interface web.

---

# Prochain module

Module 5 : Coûts et modèles frugaux.
