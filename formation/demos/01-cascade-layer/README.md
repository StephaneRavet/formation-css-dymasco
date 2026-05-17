# Démo cascade `@layer` — sanity check formateur

Démo runnable des 5 étapes du module 1. À ouvrir **la veille au soir** pour vérifier que tout marche dans le navigateur cible (Edge/Chrome récent).

## Lancer

```bash
# depuis la racine du repo
open formation/demos/01-cascade-layer/index.html
# ou via Live Server VS Code sur le même fichier
```

Le launcher renvoie vers les 5 étapes. DevTools panneau **Styles** ouvert (colonne "Layer" agrandie).

## Structure

```
01-cascade-layer/
├── README.md
├── index.html                   ← launcher
├── apriso-mini.css              ← framework hostile (NE PAS MODIFIER)
├── step-1-naive/                ← override naïf, échec spécificité
├── step-2-layer-naive/          ← @layer seul, échec unlayered
├── step-3-import-layer/         ← @import url() layer() — ça marche
├── step-4-priority/             ← couche priority pour battre !important
└── step-5-tokens/               ← variables CSS centralisées
```

## Attendus visuels

| Étape | Titres cards | Background card "alert" | Conclusion |
| --- | --- | --- | --- |
| 1 — naïf | noir (KO) | jaune (KO) | spé 0,1,0 perd contre 0,5,0 |
| 2 — `@layer` seul | noir (KO) | jaune (KO) | Apriso unlayered bat overrides layered |
| 3 — `@import layer()` | **rouge** ✅ | jaune (KO) | couches OK, mais !important framework tient |
| 4 — `priority` | **rouge** ✅ | **vert clair** ✅ | ordre couches inversé pour !important |
| 5 — tokens | **rouge** ✅ | **vert clair** ✅ | mêmes valeurs, mais centralisées |

## À vérifier dans DevTools

- **Étape 3+** : Network → `apriso-mini.css` chargé **une seule fois** (via `@import`, pas via `<link>`).
- Panneau **Styles** : colonne *Layer* visible. Survol d'une déclaration barrée → tooltip explique pourquoi elle perd.
- Onglet **Computed** : la source d'une valeur cite le nom de la couche.
- **Étape 5** : modifier la valeur de `--ml-color-danger` dans `step-5-tokens/overrides.css` → recharger → les titres suivent.

## Points de bascule (à marteler en formation)

- Étape 2 → 3 : retirer le `<link rel="stylesheet" href="apriso-mini.css">` du HTML. Sans ça, Apriso reste chargé unlayered en parallèle et reprend la main.
- Étape 3 → 4 : `@layer overrides` ne suffit pas contre un `!important` framework. Il faut une couche **dédiée** déclarée **avant** `framework`.
- Étape 4 → 5 : pas de bascule cascade, c'est juste de l'hygiène. Mais c'est ce qui rend le code maintenable sur 2 ans.

## Cible navigateur

`@layer` et `@import ... layer()` : Chrome 99+, Edge 99+, Firefox 97+, Safari 15.4+. Pour Dymasco (cible Edge ≤3 ans), zéro souci.
