---
title: "5 - TP Orchestration Multi-Agents"
weight: 2050
draft: false
---

## _Faire travailler plusieurs agents en parallèle_

> ⏱ **1h**

---

# Le problème de la session unique

Un seul agent sur une grosse tâche : contexte qui grossit, qualité qui chute, les tâches parallèles attendent. Et tout sur un gros modèle = facture qui explose.

On peut instead utiliser **un agent orchestrateur** (gros modèle) qui découpe le travail et le distribue à **des workers** (petit modèle économique) dans des environnements isolés. L'humain parle à l'orchestrateur uniquement.

---

# Architecture

```
HUMAIN
   │
   ▼
orchestrateur (Claude / gros modèle)  ← réflexion, plan, review
   │
   │  lance des workers isolés
   │
   ├──▶ worker-1 (codex -m gpt-4o-mini)  ← container / tmux pane
   ├──▶ worker-2 (codex -m gpt-4o-mini)  ← container / tmux pane
   └──▶ worker-3 (codex -m gpt-4o-mini)  ← container / tmux pane
            │
            ▼
       fichiers TASKS.md / STATUS.md / REVIEW.md
```

**Règles :**
- L'orchestrateur (gros modèle) **réfléchit** : découpe, planifie, review.
- Les workers (petit modèle) **exécutent** : pas de réflexion globale, juste appliquer une tâche cadrée.
- La communication se fait uniquement par fichiers. Chaque agent a un rôle, un contexte minimal, et une sortie définie.

---

# Étape 1 : Docker — un container par agent

Chaque worker dans son propre container : filesystem isolé, réseau contrôlable, reproductible.

## Dockerfile agent

```dockerfile
FROM node:20-slim

RUN apt-get update && apt-get install -y git && rm -rf /var/lib/apt/lists/*
RUN useradd -m agent
USER agent

WORKDIR /workspace
```

```bash
docker build -t agent-worker .
```

## Lancer un worker

```bash
docker run -dit --name worker-1 \
  -v $(pwd):/workspace \
  agent-worker \
  bash
```

Le container isole le filesystem. L'agent a **besoin** du réseau pour contacter l'API LLM, donc `--network none` n'est pas applicable ici. La protection contre l'exfiltration repose sur :

- **Pas de secrets dans le container** : pas de `.env`, pas de clés SSH, pas de `.aws/credentials`
- **User dédié sans sudo** : même si l'agent déraille, il ne peut pas lire `~/.ssh` ou `~/.aws` de la machine hôte
- **Volumes en lecture seule** quand possible : `-v $(pwd):/workspace:ro` pour les tâches qui ne modifient pas le code

```bash
docker exec worker-1 codex -m gpt-4o-mini --approval-mode full-auto "$(cat TASK_1.md)"
```

## Lancer N workers en boucle

```bash
for i in 1 2 3; do
  docker run -dit --name worker-$i \
    -v $(pwd)/task-$i:/workspace \
    agent-worker \
    bash
done
```

Chaque worker a son propre volume — pas de conflit de fichiers.

## Superviser

```bash
for i in 1 2 3; do
  echo "=== worker-$i ==="
  docker logs worker-$i --tail 20
done
```

## Nettoyer

```bash
for i in 1 2 3; do docker stop worker-$i && docker rm worker-$i; done
```

---

# Étape 2 : tmux + smux — superviser et communiquer

Docker isole, mais on ne voit pas ce qui se passe en live. tmux donne des sessions persistantes et une vue temps réel. Le skill **smux** ajoute la communication entre panes.

## Configurer smux

Installer le skill smux : <https://github.com/ShawnPana/smux>

Puis dans opencode :

```
/load-skill smux
```

## Layout tmux pour le monitoring

```bash
tmux new-session -s agents -d

# L'orchestrateur dans le pane principal
tmux send-keys -t agents 'opencode' Enter

# Split pour worker-1
tmux split-window -h -t agents
tmux send-keys -t agents 'docker exec -it worker-1 bash' Enter

# Split pour worker-2
tmux split-window -v -t agents
tmux send-keys -t agents 'docker exec -it worker-2 bash' Enter

tmux select-layout -t agents tiled
tmux attach -t agents
```

## Communication inter-agents avec tmux-bridge

`tmux-bridge` (fourni par smux) permet aux agents de communiquer entre panes :

```bash
# Lister les panes
tmux-bridge list

# L'orchestrateur envoie une tâche au worker
tmux-bridge read worker-1 20
tmux-bridge message worker-1 'Implémente la pagination dans src/api.ts'
tmux-bridge read worker-1 20
tmux-bridge keys worker-1 Enter

# Le worker répond dans le pane de l'orchestrateur
# (l'info arrive automatiquement, pas besoin de poll)
```

**Règle smux :** ne jamais poll ou attendre. L'agent destinataire répond directement dans votre pane via `[tmux-bridge from:...]`.

## Pattern orchestrateur → workers

```
1. L'orchestrateur découpe la tâche dans TASKS.md
2. Il envoie chaque sous-tâche via tmux-bridge au worker concerné
3. Chaque worker exécute dans son container, écrit son status dans STATUS.md
4. L'orchestrateur lit STATUS.md, review, et envoie des corrections si besoin
5. Résultat final dans REVIEW.md
```

---

# Étape 3 : Paseo — l'orchestration sans plomberie

Docker + tmux + smux fonctionnent, mais c'est beaucoup de plomberie manuelle. **[Paseo](https://github.com/getpaseo/paseo)** est un CLI qui orchestre plusieurs agents (Claude Code, Codex, OpenCode, Copilot) en une commande.

## Installation

```bash
npm install -g @getpaseo/cli
paseo
```

Le daemon démarre, affiche un QR code pour connecter l'app mobile/desktop.

## Lancer un agent

```bash
paseo run --provider claude/opus-4.6 "implémente l'authentification utilisateur"
```

## Lancer en parallèle avec worktrees

```bash
paseo run --provider codex/gpt-4o-mini --worktree feature-x "implémente la feature X"
paseo run --provider codex/gpt-4o-mini --worktree feature-y "implémente la feature Y"
```

## Superviser

```bash
paseo ls                           # lister les agents actifs
paseo attach abc123                # stream live output
paseo send abc123 "ajoute aussi les tests"  # follow-up
```

## Les skills Paseo

Paseo fournit des skills pour les patterns d'orchestration courants :

```bash
npx skills add getpaseo/paseo
```

| Skill | Usage |
|-------|-------|
| `/paseo-handoff` | Planifier avec un gros modèle, déléguer l'implé à un modèle économique |
| `/paseo-loop` | Boucler un agent sur des critères d'acceptation (Ralph loops) |
| `/paseo-advisor` | Lancer un agent conseiller pour un second avis |
| `/paseo-committee` | Deux agents contradictoires pour une analyse de cause racine |

**Le pattern handoff** correspond à ce qu'on a construit manuellement : Claude planifie, Codex implémente.

## Sécuriser le daemon Paseo

Paseo lance les agents en processus natifs sur votre machine — pas dans des containers. Le daemon a accès à tout ce que votre user peut faire. Quelques précautions :

- **Ne pas exposer le daemon sur le réseau public** : par défaut il écoute en local uniquement. Si vous utilisez le relay pour l'accès mobile, vérifiez que le canal est chiffré.
- **Lancer le daemon sous un user dédié** (pas votre user principal) : `sudo -u paseo paseo` — les agents n'auront accès qu'aux fichiers de ce user.
- **Configurer `paseo.json`** pour restreindre les outils disponibles :

```json
{
  "permissions": {
    "bash": "ask",
    "edit": "allow",
    "webfetch": "deny"
  }
}
```

- **Pour une isolation complète** : faire tourner le daemon Paseo lui-même dans un container Docker, avec les worktrees montés en volumes. C'est plus de setup, mais les agents n'ont aucun accès à la machine hôte.

---

# Worktrees pour vraiment paralléliser

Que vous utilisiez Docker+tmux ou Paseo, si les tâches touchent des fichiers communs, utilisez des worktrees :

```bash
git worktree add ../projet-taskN feature/task-N
```

Avec Paseo, c'est intégré via `--worktree`. Avec Docker+tmux, montez chaque worktree comme volume séparé :

```bash
docker run -dit --name worker-1 \
  -v $(pwd)/../projet-task1:/workspace \
  agent-worker bash
```

Plus aucun conflit de fichiers entre workers.

---

# Ce qu'on observe

- Le découpage en tâches atomiques par le gros modèle est crucial : un mauvais `TASKS.md` et tout s'effondre.
- Les workers `gpt-4o-mini` sont bons sur des tâches **bien cadrées** — mais s'effondrent dès qu'on leur demande de "comprendre l'archi". D'où la division du travail.
- L'orchestrateur qui review du code qu'il n'a pas écrit attrape plus de bugs qu'un agent qui se review lui-même (moins d'angle mort).
- Le coût est dominé par les tokens du gros modèle (réflexion + review), pas par les workers. Cible : 80% des tokens "code généré" sur le petit modèle.
- L'isolation Docker protège le filesystem, mais l'agent a besoin du réseau pour joindre l'API LLM — donc on compte sur l'absence de secrets dans le container plutôt que sur `--network none`.
- Paseo supprime la plupart de la plomberie tmux/docker — mais comprendre ce qui se passe en dessous reste nécessaire pour débugger.
