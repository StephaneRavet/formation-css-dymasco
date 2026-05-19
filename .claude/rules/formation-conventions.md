# Conventions — Support de formation

## Code de départ et corrigé d'un exercice

Règle : **jamais embarquer le code en bloc dans le `.md`**. Toujours un lien — autant pour le **Code de départ** que pour le **Corrigé attendu**.

| Contexte | Cible du lien |
|---|---|
| Projet local (IDE / GitHub) | Fichier source (`projet-fil-rouge/...`) |
| Notion (publication) | Page Notion dédiée contenant le code |

### Forme

- Sections : `### 📁 Code de départ` et `### Corrigé attendu`
- Lien markdown : `[chemin](<../projet-fil-rouge/...>)`
- Chemins avec espaces → wrap en `<...>`
- Plusieurs fichiers → liste à puces

### Ordre dans la section Exercice fil rouge

1. Consigne
2. Tests à faire (si présent)
3. Pièges fréquents
4. 📁 Code de départ (lien)
5. Corrigé attendu (lien)

### Exemple

```md
### 📁 Code de départ

Partez du checkpoint Module 5 : [projet-fil-rouge/checkpoints/05-selecteurs/overrides.css](../projet-fil-rouge/checkpoints/05-selecteurs/overrides.css)

### Corrigé attendu

Voir [projet-fil-rouge/checkpoints/06-container-queries/overrides.css](../projet-fil-rouge/checkpoints/06-container-queries/overrides.css).
```

### Pourquoi

- Plus de drift entre code embarqué et fichier source (cf. sync Notion qui a corrompu M01)
- `.md` plus court, plus lisible
- Un seul code source de vérité = le fichier dans `projet-fil-rouge/`

### Mapping checkpoints

| Module | Code de départ | Corrigé attendu |
|---|---|---|
| M01 | `00-base/` (HTML + `apriso-base.css` + `overrides.css` vide) | `checkpoints/01-architecture/overrides.css` |
| M02 | `checkpoints/01-architecture/overrides.css` | `checkpoints/02-typography/overrides.css` |
| M03 | `checkpoints/02-typography/overrides.css` | `checkpoints/03-flexbox/overrides.css` |
| M04 | `checkpoints/03-flexbox/overrides.css` | `checkpoints/04-grid/overrides.css` |
| M05 | `checkpoints/04-grid/overrides.css` | `checkpoints/05-selecteurs/overrides.css` |
| M06 | `checkpoints/05-selecteurs/overrides.css` | `checkpoints/06-container-queries/overrides.css` |
| M07 | `checkpoints/06-container-queries/overrides.css` | `checkpoints/07-couleurs/overrides.css` |
| M08 | `checkpoints/07-couleurs/overrides.css` | `checkpoints/08-media-queries/overrides.css` |
| M09 | `checkpoints/08-media-queries/overrides.css` | `checkpoints/09-svg-animations/overrides.css` |
| M10 | `checkpoints/09-svg-animations/overrides.css` | `checkpoints/10-perf-a11y/overrides.css` |

### Publication Notion

Lors du sync vers Notion, remplacer le lien fichier par lien vers page Notion contenant le même code. Maintenir les deux en cohérence.

Pages Notion existantes :

| Module | Page Corrigé Notion |
|---|---|
| M01 | [Corrigé Module 1](https://www.notion.so/36539bb934678196a89c0d31fcfed90b3) |
| M02 | [Corrigé Module 2](https://www.notion.so/36539bb9346781808282c3df8ee6040a43) |
| M03 | [Corrigé Module 3](https://www.notion.so/36539bb93467881b9a069f10b1a9669ef) |
| M04 | [Corrigé Module 4](https://www.notion.so/36539bb9346781d79ff1c3ab4e3b8226) |
| M05 | [Corrigé Module 5](https://www.notion.so/36539bb93467816091d1d769168a35fc) |
| M06 | [Corrigé Module 6](https://www.notion.so/36539bb9346781858fbdc5fa7a87d1f9) |
| M07 | [Corrigé Module 7](https://www.notion.so/36539bb93467810aa7d6f49068c43c0c) |
| M08 | [Corrigé Module 8](https://www.notion.so/36539bb93467810e9ffeea156a347323) |
| M09 | [Corrigé Module 9](https://www.notion.so/36539bb93467814f8bf4fecee78ed5c6) |
| M10 | [Corrigé Module 10](https://www.notion.so/36539bb934678101b6f3db9573008a67) |
