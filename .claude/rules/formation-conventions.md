# Conventions — Support de formation

## Code de départ et corrigé d'un exercice

Règle : **jamais embarquer le code en bloc dans le `.md`** support de formation. Toujours un lien — autant pour le **Code de départ** que pour le **Corrigé attendu**.

| Contexte | Cible du lien | Forme du code |
|---|---|---|
| Projet local (IDE / GitHub) | Fichier source (`projet-fil-rouge/...`) | Le fichier source EST le code |
| Notion (publication) | Page Notion dédiée | Code **intégré en code block** dans la page (le repo n'est pas publié — pas de lien `projet-fil-rouge/...` exposé au stagiaire) |

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

**Contrainte** : le repo `projet-fil-rouge/` n'est PAS pushé publiquement. Aucun lien `projet-fil-rouge/...` ne doit apparaître dans Notion — le stagiaire ne peut pas l'ouvrir.

Forme attendue d'une page corrigé Notion :

1. Titre — `Corrigé Module N — overrides.css après Module N`
2. Phrase d'intro — `État attendu de \`overrides.css\` après le **Module N — …**`
3. (Optionnel) Bullets « Ajouts vs checkpoint Module N-1 »
4. **Code block `css` contenant le contenu intégral du fichier `projet-fil-rouge/checkpoints/NN-…/overrides.css`**

Interdit dans Notion :
- Lien vers `projet-fil-rouge/...` (chemin local non navigable)
- Phrase « Le code complet est dans le fichier local … »
- Code tronqué ou résumé

### Procédure de sync Notion

À chaque modification d'un checkpoint :

1. Lire le fichier `projet-fil-rouge/checkpoints/NN-…/overrides.css` en entier
2. Mettre à jour la page Notion correspondante via MCP :
   - `mcp__claude_ai_Notion__notion-update-page` command `replace_content`
   - Contenu = intro + bullets + code block ```css contenant 100 % du fichier
3. Vérifier après update qu'aucune chaîne `projet-fil-rouge` ne subsiste dans la page

Pages Notion existantes :

| Module | Page ID |
|---|---|
| M01 | [36539bb93467819689c0d31fcfed90b3](https://www.notion.so/36539bb93467819689c0d31fcfed90b3) |
| M02 | [36539bb93467818082c3df8ee6040a43](https://www.notion.so/36539bb93467818082c3df8ee6040a43) |
| M03 | [36539bb9346781b9a069f10b1a9669ef](https://www.notion.so/36539bb9346781b9a069f10b1a9669ef) |
| M04 | [36539bb9346781d79ff1c3ab4e3b8226](https://www.notion.so/36539bb9346781d79ff1c3ab4e3b8226) |
| M05 | [36539bb93467816091d1d769168a35fc](https://www.notion.so/36539bb93467816091d1d769168a35fc) |
| M06 | [36539bb9346781858fbdc5fa7a87d1f9](https://www.notion.so/36539bb9346781858fbdc5fa7a87d1f9) |
| M07 | [36539bb93467810aa7d6f49068c43c0c](https://www.notion.so/36539bb93467810aa7d6f49068c43c0c) |
| M08 | [36539bb93467810e9ffeea156a347323](https://www.notion.so/36539bb93467810e9ffeea156a347323) |
| M09 | [36539bb93467814f8bf4fecee78ed5c6](https://www.notion.so/36539bb93467814f8bf4fecee78ed5c6) |
| M10 | [36539bb934678101b6f3db9573008a67](https://www.notion.so/36539bb934678101b6f3db9573008a67) |
