---
title: "10 - TP Orchestration Multi-Agents"
weight: 2032
draft: false
---

## _Faire travailler plusieurs agents en parallèle_

> ⏱ **1h**

---

# Le problème de la session unique

Un seul agent sur une grosse tâche : contexte qui grossit, qualité qui chute, goulot d'étranglement sur les tâches parallèles. Et tout sur un gros modèle = facture qui explose.

La solution : **un agent "cerveau"** (gros modèle, qui réfléchit et découpe) qui lance lui-même **des subagents "exécutants"** (petit modèle économique type `gpt-4o-mini`, qui appliquent le plan) dans des fenêtres tmux. L'humain ne touche jamais à tmux directement — il parle à l'orchestrateur, point.

---

# Architecture

```
HUMAIN
   │
   ▼
orchestrateur (Claude / gros modèle)  ← réflexion, plan, review
   │
   │  lance via `tmux new-window`
   │
   ├──▶ worker-1 (codex -m gpt-4o-mini)  ← exécute la tâche 1
   ├──▶ worker-2 (codex -m gpt-4o-mini)  ← exécute la tâche 2
   └──▶ worker-3 (codex -m gpt-4o-mini)  ← exécute la tâche 3
            │
            ▼
       fichiers TASKS.md / STATUS.md / REVIEW.md
```

**Règles :**
- L'humain ne crée aucune fenêtre tmux à la main.
- L'orchestrateur (gros modèle) **réfléchit** : découpe, planifie, review.
- Les workers (petit modèle) **exécutent** : pas de réflexion globale, juste appliquer une tâche cadrée.
**Règle principale :** la communication se fait uniquement par fichiers. Chaque agent a un rôle, un contexte minimal, et une sortie définie.

---

# Mise en place

## Prérequis

Un projet avec du code à refactorer ou une feature à implémenter. Utilisez votre propre projet ou le projet exemple fourni.

## Étape 1 : Ajouter le skill "smux"

<https://github.com/ShawnPana/smux>

---

## Etape 2 : demander à un premier agent de déléguer
- soit avec smux
- soit avec les subagents de Codex
- soit avec un framework dédié comme **[Hermes Agent](https://github.com/MinishLab/hermes)** pour orchestrer plusieurs agents spécialisés

---

# Worktrees pour vraiment paralléliser

Si les tâches touchent des fichiers communs, demander à l'orchestrateur d'ajouter cette consigne :

```
Avant de lancer chaque worker, crée un worktree git dédié :

  git worktree add ../projet-taskN feature/task-N

Et lance le worker DANS ce worktree :

  tmux new-window -t multi-agents -n worker-N \
    "cd ../projet-taskN && codex -m gpt-4o-mini ..."

À la fin, fusionne les branches feature/task-* dans la branche courante.
```

Plus aucun conflit de fichiers entre workers, même sur du code partagé.

---

# Ce qu'on observe

- Le découpage en tâches atomiques par le gros modèle vaut son prix : un mauvais `TASKS.md` plombe tout le reste.
- Les workers `gpt-4o-mini` sont étonnamment bons sur des tâches **bien cadrées** — mais s'effondrent dès qu'on leur demande de "comprendre l'archi". D'où la division du travail.
- L'orchestrateur qui review du code qu'il n'a pas écrit attrape plus de bugs qu'un agent qui se review lui-même (moins d'angle mort).
- Le coût total est dominé par les tokens du gros modèle (réflexion + review), pas par les workers. Cible : 80% des tokens "code généré" sur le petit modèle.

<!-- ---

# Livrable

À la fin de ce TP :

- [ ] `TASKS.md` généré par l'orchestrateur (≥ 2 tâches atomiques)
- [ ] `STATUS.md` rempli par les workers (une section par tâche)
- [ ] `REVIEW.md` rempli par l'orchestrateur avec un verdict
- [ ] `tmux list-windows -t multi-agents` montre N+1 fenêtres (1 orchestrateur + N workers)
- [ ] L'humain n'a tapé AUCUNE commande `tmux new-window` lui-même -->
