# Conventions — Support de formation

## Code de départ d'un exercice

Règle : **jamais embarquer le code en bloc dans le `.md`**. Toujours un lien.

| Contexte | Cible du lien |
|---|---|
| Projet local (IDE / GitHub) | Fichier source (`projet-fil-rouge/...`) |
| Notion (publication) | Page Notion dédiée contenant le code |

### Forme

- Section : `### 📁 Code de départ`
- Lien markdown : `[chemin](<../projet-fil-rouge/...>)`
- Chemins avec espaces → wrap en `<...>`
- Plusieurs fichiers → liste à puces

### Exemple

```md
### 📁 Code de départ

Partez du checkpoint Module 5 : [projet-fil-rouge/checkpoints/05-selecteurs/overrides.css](../projet-fil-rouge/checkpoints/05-selecteurs/overrides.css)
```

### Pourquoi

- Plus de drift entre code embarqué et fichier source (cf. sync Notion qui a corrompu M01)
- `.md` plus court, plus lisible
- Un seul code source de vérité = le fichier dans `projet-fil-rouge/`

### Mapping checkpoints

| Module | Checkpoint de départ |
|---|---|
| M01 | `00-base/` (HTML + `apriso-base.css` + `overrides.css` vide) |
| M02 | `checkpoints/01-architecture/overrides.css` |
| M03 | `checkpoints/02-typography/overrides.css` |
| M04 | `checkpoints/03-flexbox/overrides.css` |
| M05 | `checkpoints/04-grid/overrides.css` |
| M06 | `checkpoints/05-selecteurs/overrides.css` |
| M07 | `checkpoints/06-container-queries/overrides.css` |
| M08 | `checkpoints/07-couleurs/overrides.css` |
| M09 | `checkpoints/08-media-queries/overrides.css` |
| M10 | `checkpoints/09-svg-animations/overrides.css` |

### Publication Notion

Lors du sync vers Notion, remplacer le lien fichier par lien vers page Notion contenant le même code. Maintenir les deux en cohérence.
