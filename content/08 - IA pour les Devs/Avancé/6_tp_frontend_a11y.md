---
title: "6 - TP Frontend & Skills documentaires"
weight: 2060
---

## _Créer des skills qui savent lire tes docs_

> ⏱ **1h30**

---

# Principe

Un skill Codex est un fichier markdown dans `.codex/skills/`. Quand vous tapez `/nom-du-skill`, l'agent exécute les instructions du fichier.

Ce qui change ici : on va créer des skills qui **donnent à l'agent les instructions pour aller chercher le contexte lui-même** dans des documents locaux (PDF de référence, guides internes). L'agent fait le travail de conversion et d'extraction — pas vous.


# Partie 1 : Skill générique search-pdf — 20 min

Avant de créer des skills spécialisés, il faut un skill de base qui sait interroger un PDF.

## Étape 1 : Générer le skill avec $skill-creator

```
$skill-creator
```

Décrivez ce que vous voulez :

```
Crée un skill search-pdf qui :
- prend deux arguments : le chemin vers un PDF et une requête de recherche
- si le fichier .md correspondant n'existe pas, convertit le PDF avec markitdown
  (fallback : pdftotext)
- cherche la requête dans le texte extrait avec ripgrep (-A 20 -i)
- retourne les passages pertinents avec leur contexte
```

## Étape 2 : Préparer le dossier de sources

Mettez tous vos PDFs de référence dans `docs/pdfs/` — c'est ce dossier que les skills spécialisés sauront interroger.

```bash
mkdir -p docs/pdfs
curl -L https://accessibilite.numerique.gouv.fr/doc/RGAA-v4.1.pdf -o docs/pdfs/rgaa.pdf
curl -L https://owasp.org/www-project-web-security-testing-guide/assets/archive/OWASP_Testing_Guide_v4.pdf \
     -o docs/pdfs/owasp.pdf
```

## Étape 3 : Tester search-pdf

```
/search-pdf docs/pdfs/rgaa.pdf "critère 1.1"
```

```
/search-pdf docs/pdfs/owasp.pdf "SQL injection"
```

Vérifiez que l'agent extrait les bons passages sur les deux PDFs. Le skill doit fonctionner sur n'importe quel fichier sans modification.

---

# Partie 2 : Skill a11y-review — 40 min

## Contexte

Le RGAA (Référentiel Général d'Amélioration de l'Accessibilité) est le standard d'accessibilité numérique en France. C'est un PDF de ~250 pages. Les équipes front en ont besoin mais personne ne le lit en entier.

**L'idée :** un skill qui, quand vous lui passez un composant, utilise `search-pdf` pour extraire les critères RGAA pertinents et faire la review.

## Étape 1 : Générer le skill

```
$skill-creator
```

```
Crée un skill a11y-review qui :
- prend un composant en argument ($ARGUMENTS)
- lit le composant pour identifier sa nature (image, formulaire, navigation…)
- appelle /search-pdf docs/pdfs/rgaa.pdf pour extraire les critères pertinents :
  images → critères 1.x, formulaires → 11.x, couleurs → 3.x, liens → 6.x/12.x, médias → 4.x
- si d'autres PDFs dans docs/pdfs/ semblent pertinents (ex. guide WCAG),
  les interroger aussi via /search-pdf
- pour chaque critère extrait, évalue le composant : ✅ ❌ ⚠️ N/A
- écrit le rapport dans docs/a11y-report.md :
  tableau critère / statut + liste des fixes concrets avec références de lignes
```

## Étape 2 : Tester sur le composant exemple

```html
<!-- composant-exemple.html -->
<div class="card">
  <img src="banner.jpg">
  <h2>Notre offre</h2>
  <form>
    <input type="email" placeholder="votre@email.com">
    <button onclick="submit()">Envoyer</button>
  </form>
</div>
```

```
/a11y-review composant-exemple.html
```

**Ce composant a au moins 2 violations RGAA.** Le skill doit les trouver.

## Étape 3 : Appliquer sur votre vrai code

Prenez un composant réel de votre projet : formulaire, navigation, carte, page entière.

```
/a11y-review mon-vrai-composant.tsx
```

Le skill écrit le rapport dans `docs/a11y-report.md`. C'est intentionnel : **un fichier markdown est le moyen le plus simple pour qu'un agent communique avec un autre**.

Ouvrez un second terminal et demandez les fixes dans une nouvelle session :

```
Applique les conseils de ce rapport @docs/a11y-report.md
```

Re-lancez le skill pour vérifier que les violations sont corrigées.

## Optionnel : MCP Figma

Si le MCP Figma est configuré, l'agent peut lire directement les tokens de design (couleurs, typographie) et vérifier les contrastes WCAG sans screenshot ni copier-coller.

---

# Partie 3 : Skill security-review — 20 min

Même pattern. Générez le skill :

```
$skill-creator
```

```
Crée un skill security-review qui :
- prend un fichier de code en argument ($ARGUMENTS)
- appelle /search-pdf docs/pdfs/owasp.pdf pour interroger l'OWASP Testing Guide
- si d'autres PDFs dans docs/pdfs/ semblent pertinents (ex. guide de durcissement),
  les interroger aussi via /search-pdf
- identifie les catégories de risque pertinentes (auth, SQL, upload, etc.)
- évalue le code contre chaque section extraite
- écrit le rapport dans docs/security-report.md :
  nom de la vulnérabilité, référence dans le doc, ligne, fix concret
```

Lancez-le :

```
/security-review src/api/routes/users.py
```

## Bonus : appliquer des skills externes

- Sécurité : **[Claude Code Security Review](https://github.com/anthropics/claude-code-security-review)** — Skill de review sécurité pour Claude Code.
- Design : <https://github.com/vercel-labs/web-interface-guidelines>

---

##  Conclusions

- **Pas de RAG, pas de serveur** : markitdown (ou pandoc, qui convertit aussi des .docx, .epub, .html…) + ripgrep + l'agent. Ça marche en offline.
- **Les skills sont versionnés** avec le repo : toute l'équipe a les mêmes outils
- **Le contexte est précis** : l'agent reçoit exactement les critères qui s'appliquent, pas 250 pages
- **Extensible** : n'importe quel PDF de référence (OWASP, PCI-DSS, guide interne, doc d'architecture) devient interrogeable via `search-pdf`
- **Composable** : chaque agent écrit un fichier markdown, le suivant le lit — pas d'API, pas de mémoire partagée, juste des fichiers

Pour des corpus vraiment larges (> 500 pages, plusieurs livres), regarder **Docling** (IBM, open source) ou **Qdrant** pour du vrai RAG vectoriel. Mais pour la majorité des cas, ripgrep suffit.

Alternative : **[semble](https://github.com/MinishLab/semble#token-efficiency)** — BM25

> **opencode.school :** [Lesson 8 — Skills](https://opencode.school/lessons/skills/) — créer et distribuer des skills réutilisables, les partager avec l'équipe via `npx skills`.


---

# Livrable

À la fin de ce TP :

- [ ] `.codex/commands/search-pdf.md` testé sur le RGAA et sur l'OWASP Testing Guide
- [ ] `.codex/commands/a11y-review.md` qui écrit dans `docs/a11y-report.md`
- [ ] Rapport a11y sur le composant exemple avec au moins 3 violations identifiées
- [ ] Fixes appliqués depuis `docs/a11y-report.md` dans un second terminal et vérifiés avec un 2ème run
- [ ] (Bonus) `.codex/commands/security-review.md` lancé en parallèle dans un terminal séparé
