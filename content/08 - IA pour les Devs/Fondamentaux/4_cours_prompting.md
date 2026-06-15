---
title: "4 - Prompt Engineering pour Devs"
weight: 1040
---

## _Parler efficacement à l'IA_

---


En fait, la qualité de la réponse dépend de la structure de votre demande. Surtout pour des modèles plus frugaux.

---

![](../../images/ai/agile.jpeg)

---

# Les principes

## 1. Contexte explicite

- Contexte: API FastAPI avec endpoints REST.
- Problème: Le endpoint POST /users retourne 500.
- Fichier: src/api/routes/users.py, ligne 42.
- Erreur: IntegrityError sur email duplicata.
- Objectif: Ajouter une validation avant l'insertion.

---

## 2. Découper les tâches

```markdown
#❌Tout en un
"Refactor toute l'application pour ajouter l'authentification"

#✅Étapes distinctes
"Étape 1: Analyser le système d'auth actuel
Étape 2: Proposer une architecture JWT
Étape 3: Implémenter le middleware d'auth
Étape 4: Ajouter les tests"
```

En général les agents ont un outil de TODO (tâches).

---

## 3. Contraintes explicites

```markdown
#✅Contraintes claires
"- Ne pas utiliser de librairies externes
- Garder la compatibilité Python 3.11
- Préserver les tests existants
- Maximum 50 lignes ajoutées"
```

---

# Pattern : AGENTS.md

À la racine de votre projet, un fichier `AGENTS.md` donne le contexte à l'IA :

```markdown
# AGENTS.md - Contexte pour les agents IA

## Project
API REST pour gestion d'utilisateurs.

## Stack
- FastAPI 0.104+
- SQLAlchemy ORM
- PostgreSQL 15
- Pytest pour les tests

## Conventions
- Snake_case pour les variables
- docstrings Google style
- Tests dans tests/ avec pytest

## Architecture
- src/api/routes/ : endpoints REST
- src/models/ : modèles SQLAlchemy
- src/services/ : logique métier

## À NE PAS FAIRE
- Ne pas créer de nouvelles migrations DB
- Ne pas modifier .env
- Ne pas ajouter de dépendances sans validation
```

- C'est un contexte permanent
- Pas besoin de le répéter à chaque prompt
- **Plus de cohérence** dans les réponses
- Ne pas oublier de demander à l'IA de le **tenir à jour**

---

# Pattern : README.md par dossier

Chaque dossier important devrait avoir un `README.md` :

```
src/
├── api/
│   ├── README.md      # "Ce dossier contient les endpoints..."
│   └── routes/
├── models/
│   └── README.md      # "Modèles SQLAlchemy..."
└── services/
    └── README.md      # "Logique métier isolée..."
```

**Contenu type :**
```markdown
# Services Layer

Contient la logique métier isolée des controllers.

## Conventions
- Un service par domaine métier
- Injection de dépendances via __init__
- Pas d'imports circulaires

## Exemple
```python
# services/user_service.py
class UserService:
    def __init__(self, db: Session):
        self.db = db
```

---

<!-- 
## Le cas particulier du "reasoning"

Certains modèles (o3, Claude avec extended thinking…) génèrent une chaîne de réflexion interne avant de répondre. Ces tokens de raisonnement sont invisibles dans la réponse mais comptent dans la facture — et peuvent multiplier le coût par 5 sur une tâche complexe.

C'est une **feature propriétaire et opaque** : chaque provider l'implémente différemment (`thinking_budget`, `reasoning_effort`, mode automatique…), vous ne voyez pas ce qui a été raisonné, et certains outils l'activent silencieusement. À utiliser intentionnellement pour des problèmes qui le méritent, pas comme réglage par défaut. -->

---

<!-- # Anti-patterns courants

| Anti-pattern | Conséquence | Solution |
|--------------|-------------|----------|
| Prompt trop long | Confusion, tokens gâchés | Découper en étapes |
| Pas de contraintes | Code non idiomatique | Spécifier les conventions |
| Oublier les tests | Code non testé | Demander tests explicites |
| Ignorer l'existant | Duplication | Référencer les fichiers existants |

--- -->



#  Gérer son contexte — /compact et /clear

Le contexte s'accumule à chaque échange : historique de la conversation, fichiers lus, résultats de commandes, sorties de tests.

**Gérer son contexte est d'abord une affaire de focus, pas seulement de budget.** Un contexte saturé dégrade la qualité des réponses : l'agent commence à oublier des contraintes, à reproduire des erreurs déjà corrigées, à se perdre dans des chemins abandonnés. La performance chute bien avant que la limite de tokens soit atteinte.

**Règles de session :**

- **Une session = un problème.** Mélanger deux issues dans la même session pollue le contexte des deux.
- **Plusieurs sessions courtes > une longue session.** Recommencer proprement est souvent plus rapide que de gérer un agent qui dérive.
- **Commencez une nouvelle session régulièrement**, surtout après un changement de sujet ou une longue exploration.

Et chaque token d'entrée se paie à chaque nouvel échange — gérer le contexte réduit aussi les coûts.

## /compact — résumer sans perdre le fil

```
/compact
```

Ce que ça fait :
- Résume le contenu de la conversation en un bloc condensé
- Remplace l'historique détaillé par ce résumé dans le contexte
- L'agent conserve les décisions prises, les fichiers connus, l'état du projet
- Vous continuez la session sans payer pour les anciens échanges

Ce que ça ne fait **pas** :
- Ne supprime pas le contexte — `/clear` fait ça
- N'aide pas une session déjà à 90%+ — les tokens sont déjà brûlés

**Quand l'utiliser :**
- À 70% du contexte, pas après — vérifiez `Ctx(u)` dans la statusline
- Après un long passage de lecture de fichiers (tool spam)
- Avant de changer de phase : après le Plan, avant le Build

## /clear — repartir de zéro

`/clear` vide complètement le contexte. À réserver aux sessions vraiment bloquées — l'agent perd tout ce qu'il savait du projet. Préparez un résumé court à lui redonner avant de reprendre.

```bash
opencode --continue          # Reprend la dernière session compactée
```

## Seuils de contexte

| Contexte % | État | Action |
|------------|------|--------|
| 0–50% | Vert | Travaillez librement |
| 50–70% | Jaune | Soyez sélectif dans les lectures |
| 70–90% | Orange | `/compact` maintenant |
| 90%+ | Rouge | `/clear` requis |


---



# TP : Créer votre AGENTS.md

Le TP fil rouge continue : créer un `AGENTS.md` pour votre projet démo.

Voir `5_tp_agents.md` →

> **opencode.school :** [Lesson 5 — Instructions](https://opencode.school/lessons/instructions/) — guide interactif pour écrire son AGENTS.md depuis l'intérieur de l'agent.