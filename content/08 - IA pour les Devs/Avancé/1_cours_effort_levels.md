---
title: "1 - Effort & Autonomie : choisir son niveau"
weight: 2010
---

## _Quand utiliser un effort de réflexion maximal ?_

---

# Le problème

Tout le monde utilise le même réglage pour tout. Sonnet + interactif + vérification à chaque fichier pour corriger un typo. 

L'effort doit être proportionnel à la complexité et à l'impact de la tâche.


> **"Effort élevé" ≠ mode thinking.** Le *reasoning effort* est un paramètre qui dit au modèle de prendre plus ou moins de temps avant de répondre. Le "mode thinking" est plus général: il génère des tokens de raisonnement internes facturés comme des tokens de sortie. 
---

# Matrice de décision rapide

```
C'est un bug ?
├── Isolé, fichier connu → Mid
├── Intermittent, multi-système → High
└── "Je ne sais même pas d'où ça vient" → High (effort élevé)

C'est une feature ?
├── < 5 fichiers → Mid
├── > 10 fichiers, logique complexe → High pour l'architecture, Mid pour l'implémentation
└── Refacto de masse, migration →  en sandbox en autonomie

C'est une question ?
├── Syntaxe / API standard → Low
└── "Explique-moi comment marche X dans notre codebase" → Mid (l'agent lit le code)
```

---

# Le coût réel de l'effort élevé

Un cas documenté : debugging d'un race condition sur une API Node.js.

| Tentative | Niveau | Tokens | Coût | Résultat |
|-----------|--------|--------|------|----------|
| 1 | Mid (Sonnet) | 12k | $0.06 | Mauvaise piste |
| 2 | Mid (Sonnet) | 18k | $0.09 | Mauvaise piste |
| 3 | High (Sonnet ou Opus) | 45k | $0.90 | Fix correct |

**Total : $1.05 pour résoudre quelque chose qui aurait pris 3h à la main.**

Le coût n'est pas le sujet. Le sujet c'est de choisir le bon niveau au bon moment — pas de bruler des tokens High sur du Low, pas de s'obstiner en Mid quand il faut passer en High.

---

# Résumé

**Low** = question dans le vide. Rapide, pas de contexte.

**Mid** = agent dans le repo, vous supervisez. C'est là que vous passez 80% du temps.

**High** = problème dur, raisonnement profond, 3ème tentative. Intentionnel, pas par défaut.

**Async** = autonomie maximale, sandbox obligatoire, résultats au matin.

> **opencode.school :** [Lesson 11 — Agents](https://opencode.school/lessons/agents/) — basculer entre agent Plan et agent Build, et créer des agents personnalisés.
