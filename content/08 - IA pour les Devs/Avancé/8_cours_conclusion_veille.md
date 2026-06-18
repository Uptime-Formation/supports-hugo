---
title: "8 - Veille et Écosystème"
weight: 2080
---


# Le rythme effréné

**L'écosystème IA évolue très vite :**

Evolution des outils :
- Emergence d'outils asynchrones comme Jules de Google, Copilot web agents, ou **[OpenCode GUI](https://opencode.school/)** — l'équivalent open-source : on lance des tâches depuis une interface web, l'agent travaille en arrière-plan, on récupère le diff ou la PR
- De plus en plus d'outils conscients des problématiques design
- Une optimisation des coûts et de l'alternance réflexion / exécution
- Des budgets par rapport à des objectifs

---
# Le plugin oh-my-openagent et le mode Sisyphus

En mode autonome (ralph loop ou longue tâche), OpenCode peut s'arrêter silencieusement au milieu d'une session — bug connu de l'outil. Le plugin **oh-my-openagent** ajoute le mode **Sisyphus** : quand l'agent s'arrête prématurément, il est relancé automatiquement avec le contexte de la tâche.

```json
"plugin": ["oh-my-openagent"]
```

Activer le mode Sisyphus dans l'interface OpenCode avant de lancer une tâche longue sans surveillance. Sans ça, une session de nuit peut silencieusement s'arrêter à mi-chemin sans que vous vous en rendiez compte au matin.


---

# Perspectives critiques 

## Closed-source : enshittification sans préavis

Les modèles closed-source peuvent se dégrader silencieusement entre deux versions — sans changelog, sans notification. Un modèle qui était bon à l'implémentation peut devenir médiocre sur vos cas d'usage sans que vous le sachiez.


## AI Fluency Index

Selon l'[AI Fluency Index d'Anthropic](https://www.anthropic.com/research/AI-fluency-index), le mode génération d'artefacts (code, documents) crée un effet wow qui réduit l'esprit critique.

Le modèle de chat back-and-forth préserve davantage l'esprit critique.

---

# Les sources de veille

## Agrégateurs et newsletters

| Source | Fréquence | Focus |
|--------|-----------|-------|
| **Hacker News** | Quotidien | Technique, discussions |
| **Lobste.rs** | Quotidien | Technique, moins de bruit |

### Lobste.rs - Communauté technique

- Signal/bruit meilleur que HN
- Communauté plus restreinte, plus technique
- Tags : `llm`, `machine-learning`, `ai`

---

## Pour une veille avancée : LocalLLM

**r/LocalLLm** (Reddit) - La référence pour les modèles locaux :
- Benchmarks en temps réel
- Quantisation, fine-tuning, local inference
- Nouveaux modèles open source (Llama, Mistral, Qwen, etc.)
- Hardware optimisation

**Quand l'utiliser :**
- Vous voulez self-host vos modèles
- Intérêt pour les détails techniques (GGUF, quantisation)
- Tests de performance avant déploiement

---

# Les Providers et leurs produits

## Tendances à surveiller

1. **Context windows** : 200k → 1M+ tokens
2. Prix
3. Multimodal image ou non

---

# Modèles frugaux en 2026

## La guerre des prix
| Modèle | Coût/1M input tokens | Notes |
|--------|---------------------|-------|
| Gemini Flash 2.0 | $0.07 | Gratuit sur AI Studio |
| GLM-4.7 | ~$0.05 | Via OpenRouter |
| MiniMax 2.5 | ~$0.10 | Bon rapport qualité/prix |
| Claude Haiku | $0.25 | Rapide, cohérent |
| Claude Sonnet | $3.00 | Le meilleur compromis qualité | -->

- Routage intelligent selon la tâche : Haiku/Flash pour exploration et questions rapides, Sonnet pour implémentation, Opus ou extended thinking pour les cas durs.
- Tester les derniers modèles open source, souvent moins chers et largement suffisants hors cas limite
---



## Les MCP essentiels

| MCP | Usage |
|-----|-------|
| **filesystem** | Accès fichiers |
| **postgres** | Requêtes DB |
| **github** | Issues, PRs |
| **playwright** | Browser automation |
| **slack** | Messages |

---

## Parsing documentaire local

Quand vous avez des PDF de référence (OWASP, RGAA, guides internes) :

| Outil | Usage | Installation |
|-------|-------|-------------|
| **markitdown** | PDF/Word/Excel/Powerpoint → texte brut, rapide | `uvx markitdown` |
| **pdftotext** | PDF → texte brut, rapide | `apt install poppler-utils` |
| **pandoc** | PDF/Word/Excel → markdown | `apt install pandoc` |
| **ripgrep** | Chercher dans le markdown extrait | `apt install ripgrep` |
| **Docling** (IBM) | PDF complexes avec tableaux/images | `pip install docling` |

Le pattern : `pdftotext doc.pdf doc.md` → `rg "mot-clé" doc.md -A 15` → contexte donné à l'agent. Ça fonctionne offline, sans serveur, en une ligne de shell. Pour des corpus > 500 pages, regarder **Qdrant** pour du RAG vectoriel.

---

<!-- ## DeepWiki : Documentation structurée

**DeepWiki** transforme n'importe quel repo GitHub en documentation navigable :

```
Repo GitHub → DeepWiki → Markdown structuré
```

**Usage :**
- Comprendre un projet open source rapidement
- Chercher des patterns dans une codebase
- Ancrer un agent dans la documentation d'un projet

**Exemple :**
```bash
# DeepWiki pour Next.js
deepwiki fetch "vercel/next.js"
# Retourne un markdown structuré avec:
# - Architecture
# - API publique
# - Patterns utilisés
```

**Dans le workflow IA :**
```yaml
# L'agent utilise DeepWiki pour:
deepwiki_fetch:
  url: "betagouv/comparia"  # ComparIA repo
  # L'agent comprend le projet sans lire tout le code
```

**Quand l'utiliser :**
- Découvrir un projet open source
- Préparer un TP sur une techno inconnue (Rust, Elixir, etc.)
- Documenter les decisions d'architecture

--- -->
# Annexe : Liens de veille à connaître

## Outils de monitoring et d'inspection

- **[claude-devtools](https://github.com/matt1398/claude-devtools)** — Les DevTools manquants pour Claude Code : inspecter les sessions, tool calls, usage de tokens, sous-agents et fenêtre de contexte en UI visuelle.
- **[codeburn](https://github.com/AgentSeal/codeburn)** — Visualise où vont vos tokens session par session (par type de tool call, fichiers lus, etc.). Utile pour identifier ce qui consomme inutilement.
- **[rtk](https://github.com/rtk-ai/rtk)** — Proxy CLI qui réduit la consommation de tokens de 60-90% sur les commandes dev courantes.
- **[OpenChamber](https://github.com/openchamber/openchamber)** — Framework pour orchestrer et monitorer un écosystème d'agents.

## Lectures

<!-- - **[HN #47004712](https://news.ycombinator.com/item?id=47004712)** — Discussion HN à lire : retours d'expérience terrain sur l'usage des agents IA. -->
<!-- - **[Reddit ExperiencedDevs — "An AI CEO finally said something honest"](https://www.reddit.com/r/ExperiencedDevs/comments/1r6olcv/an_ai_ceo_finally_said_something_honest/)** — Analyse critique sur le discours des entreprises IA. -->
- **[Agentic Coding Trends Report 2026](https://resources.anthropic.com/hubfs/2026%20Agentic%20Coding%20Trends%20Report.pdf)** — Rapport Anthropic sur les tendances du coding agentique.
- **[AI Fluency Index](https://www.anthropic.com/research/AI-fluency-index)** — Recherche Anthropic sur l'usage réel de l'IA et le biais des artefacts.

## Répertoires de ressources

- **[Anthropic Skills](https://github.com/anthropics/skills/tree/main/skills)** — Skills officiels Claude Code : simplify, review, security-review, etc.
- **[Claude Code Security Review](https://github.com/anthropics/claude-code-security-review)** — Skill de review sécurité pour Claude Code.
- **[Claude Code Ultimate Guide](https://github.com/FlorianBruniaux/claude-code-ultimate-guide/blob/main/guide/cheatsheet.md)** — Cheatsheet community avec bonnes pratiques et quiz.