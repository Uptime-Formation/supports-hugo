---
title: "9 - TP Tool Calling et MCP"
weight: 1090
---

## _Observer ce que fait l'agent_

> ⏱ **45 min**

<!-- > **Outil principal :** Codex CLI. Remplacer `codex` par `opencode` ou `claude` selon votre outil. -->

---

# Objectif

Configurer des MCPs utiles et observer concrètement leur impact sur le comportement de l'agent.

---

# Étape 1 : Observer les tool calls


---
**Lancer l'agent :**

```bash
opencode       # tool calls visibles nativement
# ou
codex
            # Codex : tool calls visibles nativement
            # Claude Code : claude --verbose
```

**Prompt simple :**
```
>Liste tous les endpoints de l'API
```

**Observer dans les logs :**
```
[TOOL] read_file(filePath="src/api/routes.py")
[TOOL] grep(pattern="@app\.(get|post|put|delete)", output="content")
```

**Grille d'observation :**

| Critère | Oui/Non | Notes |
|---------|---------|-------|
| Lit les fichiers avant de répondre | | |
| Utilise plusieurs outils en séquence | | |
| Vérifie les résultats | | |
| Demande clarification si ambigu | | |

---

# Étape 2 : Compter les appels

**Prompt complexe :**
```
>Ajoute un endpoint GET /users/{id}/posts
>qui retourne les posts d'un utilisateur
>avec pagination (limit, offset)
```

**Tracer les appels :**
```
┌─────────────────────────────────────────────┐
│ Appel #1: read_file(src/api/routes.py)      │
│ Appel #2: read_file(src/models/post.py)     │
│ Appel #3: read_file(src/models/user.py)     │
│ Appel #4: write_file(src/api/routes.py)     │
│ Appel #5: run_command(pytest tests/)        │
└─────────────────────────────────────────────┘
```

**Question :** L'agent a-t-il modifié les bons fichiers ?

---


# Étape 3 : Recherche web — MCPs ou natif ?


**Claude Code et Codex** ont la recherche web intégrée nativement — rien à faire.

**OpenCode** ou **Roo Code** n'ont pas de recherche intégrée. Il faut ajouter un MCP comme `ddg_search`.


Opencode :
```
{
"mcp: {
"ddg_search": {
      "type": "local",
      "enabled": true,
      "command": [
        "npx",
        "-y",
        "@oevortex/ddg_search"
      ]
    }
}
}
```
---

# Étape 4 : context7 — ancrer l'agent dans la vraie doc

Quand l'agent travaille avec une librairie dont il peut avoir une connaissance périmée, context7 lui injecte la documentation réelle à jour.

**Configurer context7 :**

Opencode :
```
{
"mcp: {
"context7": {
      "type": "local",
      "enabled": true,
      "command": [
        "npx",
        "-y",
        "@upstash/context7-mcp"
      ]
    }
}
}
```

Roo Code:
> Installer via le Roo Marketplace en cliquant sur les petits cubes en haut


Codex :
```
codex mcp add context7 -- npx -y @upstash/context7-mcp
```

**Redémarrer l'agent, puis tester :**

Avec la question de votre choix :
```
>utilise context7
>Comment migrer de on_event vers lifespan dans FastAPI ?
```

**Observer le pattern :**
```
[TOOL] context7_resolve-library-id [libraryName=fastapi]
[TOOL] context7_query-docs [libraryId=/tiangolo/fastapi, query=lifespan startup shutdown]
```
L'agent répond avec la vraie API FastAPI 0.115, pas ce qu'il "croit" savoir

<!-- **Tester avec une librairie de votre choix** — n'importe quelle lib où une version récente a cassé une API connue. -->

---

# Étape 5 : Playwright — voir et interagir avec Microblog

On peut utiliser directement le MCP Chrome DevTools ou Playwright. 

**Pourquoi Playwright plutôt qu'un screenshot ?**

Playwright renvoie une représentation textuelle du DOM, pas une image. Économie de tokens massive, et le LLM peut raisonner dessus sans vision.

**Configurer le MCP Playwright :**

Codex :
```
codex mcp add playwright -- npx -y @playwright/mcp
```

Opencode :
```
{
"mcp: {
"playwright": {
      "type": "local",
      "enabled": true,
      "command": [
        "npx",
        "-y",
        "@playwright/mcp"
      ]
    }
}
}
```

**L'exercice :**

Avec Microblog qui tourne en local, demandez à l'agent d'interagir avec la page.


**Observer les appels :**
```
[TOOL] playwright_navigate(url="http://localhost:<PORT>")
[TOOL] playwright_snapshot()
[TOOL] playwright_click(ref="textarea.prompt-input")
[TOOL] playwright_fill(value="Explique le tool calling en 2 phrases")
[TOOL] playwright_click(ref="button[type=submit]")
[TOOL] playwright_snapshot()
```

> **⚠️ Codex CLI + sandbox + Playwright :** Le mode sandbox de Codex (`--approval-mode full-auto`) empêche le lancement de Playwright. Si le MCP Playwright ne répond pas sous Codex, désactivez le sandbox ou passez en mode `auto-edit`.

> **Mode headless :** Par défaut, Playwright tourne sans ouvrir de fenêtre (headless). Pour voir le navigateur s'animer en temps réel, ajoutez `"headless": false` dans la config du MCP :
> ```json
> {
>   "command": ["npx", "-y", "@playwright/mcp", "--headless=false"]
> }
> ```
> Utile pour déboguer ou montrer ce que l'agent fait visuellement.
<!-- 
# Étape 6 : LSP — navigation sémantique du code

Le cours couvre les trois situations. En pratique :

**OpenCode :** rien à faire, LSP est natif.

**Claude Code :** installez l'extension dans votre IDE (VS Code ou JetBrains). Claude Code s'y branche automatiquement.

**Codex CLI :** installez [Serena](https://github.com/oraios/serena), le MCP qui enveloppe votre language server :

```yaml
# ~/.codex/config.yaml
mcpServers:
  serena:
    command: uvx
    args: ["serena-mcp-server"]
    env:
      PROJECT_ROOT: /chemin/vers/microblog
```

**Tester la différence :**

```
# Sans LSP — grep approximatif
>Trouve toutes les fonctions qui gèrent l'authentification

# Avec LSP — navigation précise
>Trouve toutes les références à la fonction authenticate()
>et liste leurs fichiers et numéros de ligne
```

Avec LSP, l'agent ne cherche pas par mots-clés — il interroge le language server, qui connaît la structure du code.

--- -->

<!-- # Étape 7 : L'anti-pattern "tool spam"

**Observer un agent qui boucle :**

```
[TOOL] read_file(src/api/routes.py)
[TOOL] grep(pattern="auth")
[TOOL] read_file(src/api/routes.py)  # <-- Déjà lu !
[TOOL] grep(pattern="auth")           # <-- Déjà fait !
```

**Cause :** contexte trop chargé, AGENTS.md absent ou vague.
**Solution :** AGENTS.md clair sur la structure du projet, prompts structurés.

--- -->

# Étape 6 : GitHub — gh CLI ou MCP ?

Philosophie bash vs MCP. Les deux fonctionnent.

## Approche 1 — gh CLI (bash)

Si `gh` est installé et authentifié sur votre machine, l'agent peut l'utiliser directement sans aucune config :

```
>Utilise la CLI gh
>Liste les 5 dernières issues ouvertes sur miguelgrinberg/microblog
>et résume les changements de chacune
```

<!-- L'agent exécutera quelque chose comme :
```bash
gh issue create \
  --repo miguelgrinberg/microblog \
  --title "Page d'accueil lente (>3s)" \
  --body "..."
``` -->

**Avantages :** zéro config, transparent, aucune surface d'attaque supplémentaire.  
**Limite :** l'agent a accès à tout ce que `gh` peut faire — avec vos permissions complètes.

## Approche 2 — MCP GitHub

Le MCP GitHub expose des outils typés (`create_issue`, `list_pull_requests`, `get_file_contents`…) avec un périmètre configurable.

```yaml
# ~/.config/opencode/mcp.yaml
mcpServers:
  github:
    command: npx
    args: ["-y", "@modelcontextprotocol/server-github"]
    env:
      GITHUB_PERSONAL_ACCESS_TOKEN: ${GITHUB_TOKEN}
```

<!-- > Pour Claude Code : même format dans `~/.claude/settings.json` sous `mcpServers`. -->

```
>Utilise le MCP github
>Liste les 5 dernières issues ouvertes sur miguelgrinberg/microblog
>et résume les changements de chacune
```

**Avantages :** interface propre, token avec permissions fines (lecture seule si vous voulez), contexte riche.  
**Limite :** le MCP lit les issues et PRs — qui peuvent contenir des tentatives de prompt injection (voir section risques dans le cours).

## L'exercice

Faites les deux, comparez :

1. Avec `gh` : créez une issue sur un de vos repos
2. Avec le MCP : listez les PRs ouvertes et demandez un résumé

**Question :** Laquelle des deux approches vous semble plus adaptée à votre contexte ? Pourquoi ?

---



# Permissions

## Opencode


Vous pouvez définir des autorisations globalement (avec *) et remplacer des outils spécifiques.


`opencode.json` :
```json
{
  "$schema": "https://opencode.ai/config.json",
  "permission": {
    "*": "ask",
    "bash": "allow",
    "edit": "deny"
  }
}
```

ou

```json
{
  "$schema": "https://opencode.ai/config.json",
  "permission": "allow"
}
```

## OpenAI Codex CLI

```bash
# Mode suggestion (défaut) — demande approbation à chaque action
codex "ajoute la pagination"

# Auto-edit — approuve les modifications de fichiers, demande pour les commandes shell
codex --approval-mode auto-edit "$(cat TASK.md)"

# Full-auto — approuve tout, y compris les commandes shell arbitraires
codex --approval-mode full-auto "$(cat TASK.md)"
```

`full-auto` ne s'utilise qu'à l'intérieur d'un sandbox. Jamais sur votre machine principale.

## Claude Code

```bash
# Mode interactif normal — Claude demande avant chaque action sensible
claude

# Skipper TOUTES les permissions — à n'utiliser qu'en sandbox
claude --dangerously-skip-permissions

```

`--dangerously-skip-permissions` approuve automatiquement : lecture/écriture de fichiers, exécution de commandes shell, appels réseau. Le nom est volontairement alarmant.


---

# Bonus : autres MCPs utiles

Une fois que vous êtes à l'aise avec le pattern MCP, voici ce que la communauté utilise le plus.

## Communication

**Slack**
```yaml
slack:
  command: npx
  args: ["-y", "@modelcontextprotocol/server-slack"]
  env:
    SLACK_BOT_TOKEN: ${SLACK_BOT_TOKEN}
    SLACK_TEAM_ID: ${SLACK_TEAM_ID}
```
L'agent peut envoyer des messages, lire des canaux, rechercher dans l'historique. Utile pour des notifications automatiques en fin de tâche.

**Telegram**  
Plusieurs MCPs communautaires disponibles — pratique si votre équipe est sur Telegram plutôt que Slack.

**Filesystem étendu** — accès à des dossiers en dehors du projet courant (logs système, exports, etc.).

**Rappel sécurité :** Un package compromis s'exécute avec vos permissions.

<!-- 
# Livrable

À la fin de ce TP :

- [ ] Avoir observé et compris les tool calls dans les logs
- [ ] context7 configuré et testé sur une librairie réelle
- [ ] Playwright : Microblog chargée et interagie via l'agent
- [ ] LSP configuré (ou compris pourquoi c'est déjà là)
- [ ] GitHub : les deux approches testées (gh CLI + MCP)
- [ ] Avoir identifié un anti-pattern tool spam dans vos observations -->

---

# Prochain module

Module 10 : Skills & Tests visuels.
