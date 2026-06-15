---
title: "2 - Workflow Git & Docker"
weight: 2020
---

**Les agents LLM modifient votre code.**  
Sans workflow structuré, vous perdez :
- L'historique des changements
- La possibilité de revenir en arrière
- La traçabilité de ce que l'agent a fait


---

# Git Workflow Obligatoire

## Règle #1 : Une feature = Une branche

```bash
# JAMAIS travailler sur main/master directement
git checkout main
git pull origin main
git checkout -b feature/nom-de-la-feature
```

**Pourquoi ?**  
- Isolation des changements
- Possibilité d'abandonner sans casser main
- Historique propre

---

## Règle #2 : Commits atomiques

**Un commit = Une logique.**

```bash
# BON : commits atomiques
git add src/auth.ts
git commit -m "feat: add JWT token validation"

git add src/middleware.ts
git commit -m "feat: add auth middleware"

git add tests/auth.test.ts
git commit -m "test: add auth JWT tests"

# MAUVAIS : un gros commit
git add .
git commit -m "feat: add auth system with tests and fixes and refactors"
```

**Comment écrire un bon message :**

```
<type>: <description courte>

Types possibles:
- feat: nouvelle fonctionnalité
- fix: correction de bug
- refac: refactoring
- test: ajout de tests
- docs: documentation
- chore: tâche maintenance
```

---

## Règle #3 : Agent Visibility

**Git est votre fenêtre sur ce que l'agent fait.**

```bash
# Avant de demander à l'agent
git status  # Vérifier l'état

# Après que l'agent a fait des changements
git diff    # Voir exactement ce qui a changé

# Si l'agent modifie 50 fichiers
git diff --stat  # Vue d'ensemble

# Isoler les changements par fichier
git diff src/specific-file.ts
```

**Pattern de sécurité :**

```markdown
YOU: "Add authentication to the app"
AGENT: [modifies 12 files]

YOU: 
git diff --stat
# Hmm, pourquoi setup.py a changé ?

git diff setup.py
# L'agent a ajouté une dépendance suspicieuse ?

# Questionner l'agent avant de commit
```

---

## Règle #4 : Push régulier

```bash
# Push à chaque milestone
git push origin feature/ma-feature

# En cas de problème majeur
git reset --hard origin/feature/ma-feature  # Revenir au dernier push
```

---

## Règle #5 : Slop Removal (Avant Commit)

**Les agents génèrent du code "slop" - code inutile qu'il faut nettoyer.**

### Qu'est-ce que le slop ?

| Type | Exemple | Pourquoi c'est un problème |
|------|---------|---------------------------|
| **Commentaires TODO** | `// TODO: improve this` | Reste dans le code |
| **Code commenté** | `// function oldVersion()` | Pollue le code |
| **Imports inutilisés** | `import { unused }` | Bundle size |
| **Console.log** | `console.log("debug")` | Logs en production |
| **Fonctions mortes** | `function oldHelper()` | Maintenance burden |
| **Types inutilisés** | `interface OldFormat` | Confusion |


## Règle #6 : Apprendre à reconnaître les patterns d'échec et à les corriger.

## Indicateurs à monitorer

| Indicateur | Bon |Mauvais |
|------------|-----|--------|
| Fichiers modifiés | 5-10 | 50+ |
| Tokens par itération | Stable | Croissant |
| Tests passant | ↑ | Stagnant |
| Temps par itération | Stable | Croissant |

## Les 8 patterns à connaître

| Pattern | Signal | Fix |
|---------|--------|-----|
| **Hallucinated API** | Imports qui n'existent pas | AGENTS.md liste les utilitaires disponibles |
| **Infinite Fix Loop** | A→casse B→fixe B→casse A | Limite d'itérations + tests de régression |
| **"Done" Bug** | "C'est fait !" mais 0 fichiers modifiés | Vérifier indépendamment avec `git diff --stat` |
| **Ignoring Directives** | Modifie .env malgré l'interdiction | Répéter les contraintes, watchdog externe |
| **Tool Spam** | Lit le même fichier 5 fois | AGENTS.md structuré, contexte clair |
| **Context Amnesia** | Oublie ce qui a été décidé avant | Rotation à 70%, commits fréquents |
| **Destructive Edits** | Supprime des fichiers "inutiles" | Fichiers protégés dans AGENTS.md |
| **Type Error** | Passe une fonction au lieu d'une valeur | Tests de type, CI strict |

**Règle d'or :** Ne demandez pas à l'agent si c'est fait, `make test` et `git diff`.

### Pattern de cleanup obligatoire


```bash
# APRÈS que l'agent a fini, AVANT de commit
git diff --stat
# Vérifier quels fichiers sont modifiés

# Identifier le slop
git diff src/new-feature.ts
# L'agent a-t-il laissé des console.log ?
# Des commentaires TODO ?
# Des imports inutilisés ?

# Demander le cleanup
YOU: "Remove all console.log, commented code, and unused imports from src/new-feature.ts"

# Re-vérifier
git diff src/new-feature.ts

# Maintenant commit propre
git add src/new-feature.ts
git commit -m "feat: add new feature"
```

### Slop Removal Checklist

```markdown
Avant chaque commit, vérifier :
- [ ] Pas de console.log/print statements
- [ ] Pas de code commenté
- [ ] Pas de TODO/FIXME laissés
- [ ] Imports inutilisés supprimés
- [ ] Fonctions mortes supprimées
- [ ] Variables inutilisées supprimées
```

**A mettre dans `AGENTS.md` typiquement

---

## Règle #6 : Commits Multiples par Feature

**Une feature = Plusieurs commits atomiques.**

```markdown
YOU: "Add user authentication. Commit after each logical step."

AGENT: 
  1. Creates auth types
  2. git add src/types/auth.ts && git commit -m "feat: add auth types"
  
  3. Adds JWT validation
  4. git add src/auth/jwt.ts && git commit -m "feat: add JWT validation"
  
  5. Creates middleware
  6. git add src/middleware/auth.ts && git commit -m "feat: add auth middleware"
  
  7. Adds tests
  8. git add tests/auth.test.ts && git commit -m "test: add auth tests"

YOU: git log --oneline -5
# 5 commits reviewables, rollbackables
```

### Pattern d'orchestration

```markdown
YOU: "Implement [FEATURE]. Make atomic commits after each logical step:
     1. Types/interfaces first
     2. Core implementation
     3. Integration/middleware
     4. Tests
     5. Documentation
     
     Run slop removal before each commit."

AGENT: [Implements step 1]
       [Removes slop]
       [Commits]
       [Implements step 2]
       ...
```

---

## Le pattern sécurisé

```markdown
✅ GOOD:
YOU: "Fix the bug in auth.ts. Only touch auth.ts."

[AGENT modifies auth.ts]

YOU: git diff auth.ts
# Vérifiez les changements

YOU: "Why did you change line 42?"

AGENT: Explains the reasoning

YOU: "OK, commit that separately from the fix."

git add auth.ts
git commit -m "fix: auth token expiration check"
```

---

## Catastrophic Forgetting

**⚠️ Si l'agent n'a pas de guardrails intégrés, l'oubli catastrophique contournera vos protections.**

**Ce qui arrive :**
1. Vous dites "Don't modify files outside src/"
2. L'agent dit "OK"
3. 20 messages plus tard...
4. L'agent "oublie" et modifie `package.json`
5. Vous ne remarquez pas

**Solution : Le test du nom**

```
Dans votre AGENTS.md (ou instructions agent):

"Call me by my name at the beginning of each message.
 If you forget my name, it means you're experiencing
 catastrophic forgetting and should compress context."

Agent: "Hadrien, here's the fix..."

Agent: "Here's another fix..."  ← ALERTE : forgetting!

Vous: /compact  ← Compresser le contexte
```

**Pourquoi ça fonctionne :**  
L'instruction "call me by name" est en préfixe du contexte.  
Si le contexte grossit trop, cette instruction est éjectée.  
L'agent arrête de vous nommer = signal d'alerte.

---

# MCP Web Search (Transversal)

## Pourquoi MCP Search ?

**Les agents ont besoin de rechercher hors du code.**

- Documentation externe
- Solutions sur Stack Overflow
- Bugs connus dans les issues GitHub
- Nouvelles versions de packages

**MCP Search = Interface unifiée.**
**Brave Search**: Alternative privacy-first 

---

## Installation (exemple Brave Search)

```json
// mcp-config.json
{
  "mcpServers": {
    "brave-search": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-brave-search"],
      "env": {
        "BRAVE_API_KEY": "your-key-here"
      }
    }
  }
}
```

