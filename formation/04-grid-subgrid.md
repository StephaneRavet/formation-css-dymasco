---
formateur:
  prerequis_formateur:
    - savoir écrire grid-template-areas + grid-area en live
    - connaître les 4 conditions de position: sticky (offset, ancêtre scrollable, pas overflow:hidden, background)
    - savoir expliquer display: contents (élément invisible au layout, enfants prennent sa place)
    - différence auto-fit (collapse vides) vs auto-fill (conserve vides)
    - Subgrid widely available mi-2024 — vérifié sur cible Dymasco
  ressources_demo_a_preparer:
    - checkpoint Module 3 ouvert
    - DevTools panneau "Layout" pour montrer overlay Grid (areas en surimpression)
    - tableau récap auto-fit/auto-fill imprimé
    - gridbyexample.com en favori
  pitch_ouverture: >
    "Module le plus long de la formation, 2h15. Mais c'est celui qui débloque le plus
    de douleurs quotidiennes : transformer un <table> qu'on subit, faire un sticky
    header qui marche, et la cerise — Subgrid pour aligner des sous-éléments entre
    composants."
  energie_attendue: haute pour démarrer, prévoir mini-pause à mi-parcours (1h passé)
  duree_cible: 135 min — découpage 45 concepts / 25 démo / 50 exo / 15 récap (avec démo Subgrid clôture 5 min)
  variantes_timing:
    si_en_retard: zapper bonus place-items + grid-auto-flow dense + min-content/max-content
    si_en_avance: faire écrire un layout complet à voix haute par les apprenants
  points_a_marteler:
    - "display: contents fait DISPARAÎTRE l'élément du flux layout (table → grid)"
    - "position: sticky a 4 conditions strictes — manquer 1 = silence total"
    - "JAMAIS de sticky sans background opaque (sinon transparent, contenu défile derrière)"
    - "scrollbar-gutter: stable = layout qui ne saute plus à l'apparition scrollbar"
    - "Subgrid : démo 5 min, sans exo — message c'est qu'il EXISTE"
  pieges_apprenants:
    - display: contents qui casse styles hérités (bordures, bg du <table> perdus)
    - sticky qui ne colle nulle part → ancêtre overflow: hidden quelque part
    - confondre auto-fit (vide collapse) et auto-fill (vide conservé) → items étirés non voulu
    - 1fr sans minmax(min, 1fr) → cards illisibles à très petit écran
    - oublier le background sur sticky → contenu défile derrière, illisible
  questions_probables:
    - q: "display: contents et accessibilité ?"
      r: bug arbre a11y corrigé depuis 2023. Vérifier sur cible client (Module 0).
    - q: "Grid vs Flex pour une grille de cards ?"
      r: Grid si alignement strict colonnes. Flex pour wrap libre tags/breadcrumbs.
    - q: "Subgrid quand vraiment ?"
      r: aligner sous-éléments (header/metrics/footer) ENTRE cards. Rare mais magique.
    - q: "Combien de sticky dans une même page ?"
      r: pas de limite tech. Empilable via top: 0 puis top: 32px.
  transition_module_suivant: >
    "Layout maîtrisé. Mais le code commence à charger — sélecteurs longs, répétitions.
    Module 5 : sélecteurs modernes :is() :where() :has() — moins de code, plus de
    puissance."
---

# Module 4 — Grid Layout & layouts denses

> ⏱️ **Durée** : 2h15 — **J1 après-midi, gros bloc**
> 🧭 **Type** : module fondateur (Grid, sticky, scrollbar-gutter, tableaux denses + démo Subgrid de clôture)
> 🎯 **Checkpoint** : `projet-fil-rouge/checkpoints/04-grid/overrides.css`

---

## 🎯 Objectif (en 1 phrase)

Reprendre le contrôle du **layout 2D** du dashboard avec Grid, transformer un `<table>` rigide en grille fluide, fixer les en-têtes de tableau scrollable, et réussir des **tableaux denses lisibles**.

---

## ⚙️ Pour qui c'est utile chez Dymasco

Les dashboards d'atelier sont **denses** : KPI, grilles de machines, journaux scrollables, footers. Tout est en 2D, beaucoup en tableau, et tout doit rester lisible **sans scroll horizontal**.

L'arsenal de ce module attaque trois douleurs récurrentes :

1. **Le `<table>` qu'on subit** (côté Apriso, le KPI Board est en `<table>` HTML — pas modifiable). On le **transforme en Grid** par CSS pur.
2. **Le journal scrollable qui perd son en-tête** quand on descend dans la liste. → `position: sticky`.
3. **Le layout qui saute** quand une scrollbar apparaît. → `scrollbar-gutter`.

Aujourd'hui dans `apriso-base.css` :

```css
.apriso-kpi-board__table { width: 100%; table-layout: fixed; }   /* rigide */
.apriso-event-log__table thead th { /* PAS sticky */ }
.apriso-event-log__container { overflow-y: auto; /* PAS de scrollbar-gutter */ }
```

---

## 📚 Concepts (45 min)

### 1. Rappel express Grid (5 min)

Grid = **2D** (lignes ET colonnes), contrairement à Flex (1D + wrap).

```css
.parent {
  display: grid;
  grid-template-columns: 200px 1fr 200px;   /* 3 colonnes */
  grid-template-rows: 64px 1fr 40px;        /* 3 lignes */
  gap: 16px;
}
```

Unités spécifiques Grid :

| Unité | Sens |
| --- | --- |
| `1fr` | "1 part de l'espace restant" |
| `auto` | Taille du contenu |
| `min-content` / `max-content` | Plus petit / plus grand contenu |
| `minmax(min, max)` | Borne min et max |

Cible Baseline : Grid = **Widely available** depuis ~ 2017. ✅.

### 2. `grid-template-areas` — le layout en ASCII art (10 min)

Le moyen le plus lisible de décrire un layout 2D nommé.

```css
.apriso-dashboard {
  display: grid;
  grid-template-rows: auto auto 1fr auto;
  grid-template-columns: 1fr 380px;
  grid-template-areas:
    "header  header"
    "kpi     kpi"
    "main    sidebar"
    "footer  footer";
}

.apriso-header  { grid-area: header; }
.apriso-kpi-board { grid-area: kpi; }
.apriso-machine-grid { grid-area: main; }
.apriso-event-log { grid-area: sidebar; }
.apriso-footer  { grid-area: footer; }
```

**Lecture immédiate** : on voit la structure dans le CSS. Réorganiser = déplacer un nom dans la chaîne.

> 💡 **Astuce DevTools** : Chrome/Edge affichent le nom des areas en surimpression quand on hover sur le conteneur grid. Très pédagogique en démo.

### 3. `auto-fit` / `auto-fill` + `minmax()` — la grille qui respire (10 min)

Le pattern Grid **moderne** pour des cards qui s'adaptent **automatiquement** au contexte.

```css
.apriso-machine-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: var(--ml-spacing-3);
}
```

Lecture : "Crée autant de colonnes que possible, chacune entre 240px et 1fr."

#### `auto-fit` vs `auto-fill`

| Valeur | Comportement |
| --- | --- |
| `auto-fill` | Crée toutes les colonnes possibles, **garde les vides** s'il en reste. |
| `auto-fit` | Idem, mais **collapse les colonnes vides** → les items remplissent l'espace. |

Pour un dashboard : **`auto-fit` quasi systématique**. `auto-fill` utile si on veut garder un alignement régulier "comme s'il y avait toujours 4 colonnes".

#### Comparaison avec Flex (Module 3)

| Approche | Code | Avantage |
| --- | --- | --- |
| Flex | `flex: 1 1 240px` | Simple, items s'étirent |
| Grid `auto-fit` | `repeat(auto-fit, minmax(240px, 1fr))` | Items strictement alignés en colonnes, gap parfait |

→ Pour des cards alignées en grille régulière : **Grid gagne**. Flex reste meilleur pour des layouts inline avec wrap (tags, breadcrumbs).

### 4. Transformer un `<table>` HTML en Grid (5 min)

Cas Apriso : le KPI Board est un vrai `<table>`, on n'a pas la main sur le HTML.

**Solution** : sortir les éléments table du flux table avec `display: contents` ou `display: grid`.

```css
.apriso-kpi-board__table,
.apriso-kpi-board__table tbody {
  display: contents;
}

.apriso-kpi-board__table tbody tr {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: var(--ml-spacing-4);
}
```

**Comment ça marche** :

- `display: contents` rend l'élément **invisible au layout** : ses enfants prennent sa place dans la cascade de layout.
- `<table>` et `<tbody>` "disparaissent". Le `<tr>` devient le conteneur grid direct, ses `<td>` deviennent les items.

> 🚫 **Quand refuser `display: contents` sur `<table>`** : si le tableau porte de **vraies données tabulaires** (export CSV, lecture lecteur d'écran ligne par ligne, tri colonne) → ne **jamais** casser la sémantique table. Réservé aux tableaux purement présentationnels (KPI board, layout détourné). En cas de doute → garder `<table>` natif et styler avec `border-collapse` / `table-layout: fixed`.

> 💡 **Note pédagogique — Subgrid** : la propriété `subgrid` (aligner les enfants d'une card sur la grille du parent) est **volontairement sortie du tronc commun**. Le module est déjà dense (Grid + areas + auto-fit + display:contents + sticky + scrollbar-gutter + tableaux denses). Subgrid est traité en **démo de clôture de 5 min**, sans exercice — voir § *Pour aller plus loin* en bas de module.

### 5. `position: sticky` — l'en-tête qui suit (8 min)

```css
.apriso-event-log__table thead th {
  position: sticky;
  top: 0;
  background: var(--ml-color-bg-cream);  /* IMPORTANT — sinon transparent */
  z-index: 1;
}
```

**Comment ça marche** : l'élément reste dans le flux **jusqu'à** ce qu'il atteigne `top: 0` du conteneur scrollable, puis se "colle" tant qu'on scrolle.

#### Conditions pour que ça marche

1. L'élément a `position: sticky` ET une **valeur d'offset** (`top`, `bottom`, `left`, `right`).
2. L'**ancêtre scrollable** (`overflow: auto`) est le conteneur de référence.
3. L'élément ne doit **pas** être dans un parent avec `overflow: hidden` non scrollable.
4. **Background opaque obligatoire** : sticky ne masque pas ce qui passe derrière, on voit à travers.

#### Cas du `<thead>` complet

Sticky **par cellule**, pas par ligne. Pour rendre toute une ligne `<tr>` sticky :

```css
.apriso-event-log__table thead th {
  position: sticky;
  top: 0;
  /* idem pour toutes les cellules */
}
```

→ Chaque `<th>` se colle individuellement, ce qui donne l'illusion d'une ligne sticky.

> 💡 Pour empiler une "ligne 1" et "ligne 2" sticky : `top: 0` puis `top: 32px` (la hauteur de la première).

### 6. `scrollbar-gutter` — le layout qui ne saute plus (4 min)

Sur Windows et Linux, les scrollbars **prennent de la place**. Quand le contenu déborde, le layout se décale brutalement (~16px) à l'apparition de la scrollbar.

```css
.apriso-event-log__container {
  overflow-y: auto;
  scrollbar-gutter: stable;
}
```

→ Le navigateur **réserve la place** de la scrollbar même si elle n'est pas visible. Plus de saut.

Variante :

```css
scrollbar-gutter: stable both-edges;   /* symétrie visuelle */
```

Cible Baseline : **Widely available** depuis 2022. ✅.

### 7. Patterns tableaux denses (3 min)

Pour un journal d'événements lisible, retenir 4 règles :

#### a) `tabular-nums` sur les colonnes numériques / heures

```css
.apriso-event-log__table td:first-child {   /* colonne Heure */
  font-variant-numeric: tabular-nums;
  text-align: right;          /* ou start, selon convention métier */
}
```

→ Tous les chiffres ont la **même largeur** → colonnes alignées au pixel.

#### b) Zebra rows propre

```css
:where(.apriso-event-log__table tbody tr:nth-child(even)) {
  background: var(--ml-color-bg-cream);
}
```

`:where()` pour ne pas escalader la spécificité (cf. Module 1).

#### c) Hover discret

```css
.apriso-event-log__table tbody tr:hover {
  background: color-mix(in oklab, var(--ml-color-brand-accent), white 88%);
}
```

#### d) Alignement métier

| Type de colonne | `text-align` | `font-variant-numeric` |
| --- | --- | --- |
| Heure / horodatage | `end` ou `start` | `tabular-nums` |
| Code / référence | `start` | — |
| Libellé long | `start` | — |
| Sévérité (badge) | `end` | — |

---

## 🔍 Démonstration (25 min)

**Live** : refondre **3 zones du dashboard** une à une.

### Étape 1 — Layout du dashboard via `grid-template-areas` (8 min)

Le base utilise déjà Grid mais sans areas nommées. Réécrire avec areas. Modifier un area dans le code → la zone bouge à l'écran.

Effet pédagogique : "le CSS **ressemble** au layout".

### Étape 2 — KPI Board : `<table>` → Grid (8 min)

Avant : KPI alignés via `display: table`. Pas modifiable côté HTML.

```css
@layer overrides {
  .apriso-kpi-board__table,
  .apriso-kpi-board__table tbody {
    display: contents;
  }
  .apriso-kpi-board__table tbody tr {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: var(--ml-spacing-4);
  }
}
```

→ KPI affichés en Grid 4 colonnes égales. DevTools : montrer la disparition du `<table>` du flux layout.

### Étape 3 — Sticky header journal + scrollbar-gutter (5 min)

Scroller le journal **avant** modif → en-tête disparaît. Ajouter sticky + bg → en-tête reste collé. Ajouter `scrollbar-gutter: stable` → plus de saut quand on filtre la liste.

### Étape 4 — Tableau journal stylé (4 min)

Ajouter `tabular-nums` sur la colonne Heure, zebra `:nth-child(even)`, hover discret. Effet "données pro" immédiat.

---

## 🛠️ Exercice fil rouge (50 min)

### Consigne

Reprendre le checkpoint Module 3 et :

1. **Layout dashboard** : réécrire `.apriso-dashboard` en utilisant `grid-template-areas` (4 zones : header, kpi, main, footer ; `main` lui-même est split en `machines + sidebar`).
2. **Grille machines** : remplacer le `flex: 1 1` du Module 3 par :
	```css
.apriso-machine-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(var(--ml-card-width-min), 1fr));
  gap: var(--ml-spacing-3);
  padding: var(--ml-spacing-4);
}
	```
	Et **retirer** `flex` / `width` / `max-width` sur la card.
3. **KPI Board** : transformer le `<table>` en Grid 4 colonnes via `display: contents` (cf. concept §4).
4. **Journal d'événements** :
	- `<thead> th` → `position: sticky; top: 0; background; z-index`.
	- `.apriso-event-log__container` → `scrollbar-gutter: stable`.
	- Première colonne (Heure) → `tabular-nums`, alignement `end`.
	- Dernière colonne (Sévérité) → alignement `end`.
	- Lignes paires → background zebra (token `--ml-color-bg-cream`).
	- Hover discret sur les lignes.

### Tests à faire

- Resize la fenêtre : grille machines doit reflow proprement (autant de colonnes que possible).
- Scroller le journal : en-tête doit rester visible.
- Filtrer mentalement la liste : vérifier qu'aucun saut horizontal n'apparaît.
- Tester les cards avec un nombre d'items varié (4, 6, 8, 12…).

### Pièges fréquents

- ❌ `display: contents` cassant le style hérité (bordures, backgrounds du `<table>` perdus). Solution : redéfinir explicitement sur les enfants.
- ❌ `position: sticky` sans `background` → on voit le contenu défiler **derrière** l'en-tête, illisible.
- ❌ `position: sticky` sans **conteneur scrollable** identifiable → ne se colle nulle part.
- ❌ Mettre `overflow: hidden` sur un ancêtre → casse le sticky silencieusement.
- ❌ Confondre `auto-fit` (collapse vides) et `auto-fill` (conserve vides) → grille avec items étirés non voulu.
- ❌ Utiliser `1fr` sans `minmax(min, 1fr)` → items qui peuvent devenir illisibles à très petit écran.

### 📁 Code de départ

Partez de ce `overrides.css` (état après Module 3) :

```css
/* ============================================================================
   FORGE × PÂTES MAMIE LULU — overrides.css
   ----------------------------------------------------------------------------
   Checkpoint après Module 3 — Flexbox avancé + textes débordants
   Cible : Edge/Chromium <= 3 ans (Baseline widely available OK)
   Last reviewed : 2026-05-18
   ----------------------------------------------------------------------------
   Ajouts vs checkpoint Module 2 :
   - Header en pattern fixe + fluide + fixe (flex 0 0 auto / 1 1 auto / 0 0 auto)
   - Cards machine en flex: 1 1 var(--ml-card-width) — remplissent l'espace
   - Triptyque textes : ellipsis 1 ligne, line-clamp 2, overflow-wrap anywhere
   - min-width: 0 systématique sur les ancêtres flex de textes ellipsés
   ============================================================================
*/

@layer reset, priority, framework, overrides;

/* Apriso amené dans la cascade via une couche nommée. Le <link> direct vers
   apriso-base.css doit être retiré du HTML. Voir Module 1 §3. */
@import url("./apriso-base.css") layer(framework);

/* ----------------------------------------------------------------------------
   PRIORITY — escalade !important pour battre les 6 !important Apriso
   ---------------------------------------------------------------------------- */

@layer priority {
    .apriso-header__title { font-size: var(--ml-fs-header-title) !important; }
    .apriso-machine-card--running     { border-left-color: var(--ml-color-status-running) !important; }
    .apriso-machine-card--idle        { border-left-color: var(--ml-color-status-idle) !important; }
    .apriso-machine-card--maintenance { border-left-color: var(--ml-color-status-maintenance) !important; }
    .apriso-machine-card--alert {
        border-left-color: var(--ml-color-status-alert) !important;
        background: color-mix(in oklab, var(--ml-color-status-alert), white 90%) !important;
    }
}

/* ----------------------------------------------------------------------------
   RESET
   ---------------------------------------------------------------------------- */

@layer reset {
    :where(h1, h2, h3, h4, h5, h6, p, dl, dd, dt) {
        margin: 0;
    }

    :where(button, select, input, textarea) {
        font: inherit;
        color: inherit;
    }
}

/* ----------------------------------------------------------------------------
   OVERRIDES
   ---------------------------------------------------------------------------- */

@layer overrides {

    /* --- TOKENS ----------------------------------------------------------- */

    :root {
        /* Brand */
        --ml-color-brand-primary: #c2410c;
        --ml-color-brand-accent:  #f59e0b;
        --ml-color-bg-cream:      #fef6e4;
        --ml-color-bg-app:        #faf3e0;
        --ml-color-text:          #1f1f1f;
        --ml-color-text-muted:    #6b5d4f;
        --ml-color-border:        #e5d4b1;
        --ml-color-surface:       #ffffff;

        /* Status */
        --ml-color-status-running:     #2e7d32;
        --ml-color-status-idle:        #6c757d;
        --ml-color-status-alert:       #f57c00;
        --ml-color-status-maintenance: #1976d2;

        /* Spacing */
        --ml-spacing-1: 4px;
        --ml-spacing-2: 8px;
        --ml-spacing-3: 12px;
        --ml-spacing-4: 16px;
        --ml-spacing-5: 24px;

        /* Typo fluide */
        --ml-fs-base:        clamp(13px, 0.95vw, 15px);
        --ml-fs-header-title: clamp(16px, 1.6vw, 24px);
        --ml-fs-kpi-label:   clamp(10px, 0.9vw, 13px);
        --ml-fs-kpi-value:   clamp(20px, 2.4vw, 40px);
        --ml-fs-card-title:  clamp(13px, 1vw, 16px);

        /* Dimensions */
        --ml-dashboard-max-width: 1920px;
        --ml-card-width:          clamp(220px, 22vw, 300px);
        --ml-card-max:            360px;
        --ml-select-width:        max(180px, 14ch);

        /* Radius / shadow */
        --ml-radius-sm: 4px;
        --ml-radius:    8px;
        --ml-shadow-1:  0 1px 2px rgb(0 0 0 / 0.06);
        --ml-shadow-2:  0 4px 12px rgb(0 0 0 / 0.10);
    }

    /* --- APP -------------------------------------------------------------- */

    html {
        background: var(--ml-color-bg-app);
        color: var(--ml-color-text);
        font-size: var(--ml-fs-base);
    }

    .apriso-dashboard {
        width: min(var(--ml-dashboard-max-width), 100%);
        height: 100dvh;
        background: var(--ml-color-surface);
        border-color: var(--ml-color-border);
    }

    /* --- HEADER — pattern fixe + fluide + fixe ---------------------------- */

    .apriso-header {
        display: flex;
        align-items: center;
        gap: var(--ml-spacing-4);
        padding-left: max(16px, 2vw); padding-right: max(16px, 2vw);
        background: var(--ml-color-brand-primary);
        border-bottom-color: color-mix(in oklab, var(--ml-color-brand-primary), black 15%);
    }

    .apriso-header__brand    { flex: 0 0 auto; }
    .apriso-header__controls { flex: 0 0 auto; }

    .apriso-header__title {
        /* font-size : escaladé dans @layer priority (Apriso le pose en !important) */
        flex: 1 1 auto;
        min-width: 0;
        text-wrap: balance;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
    }

    .apriso-line-selector__select {
        width: var(--ml-select-width);
    }

    .apriso-connection-status__dot {
        width: 0.6em;
        height: auto;
        aspect-ratio: 1;
    }

    /* --- KPI BOARD -------------------------------------------------------- */

    .apriso-kpi-board {
        background: var(--ml-color-bg-cream);
        border-bottom-color: var(--ml-color-border);
        padding: min(12px, 1.5vw) min(16px, 2vw);
    }

    .apriso-kpi__label { font-size: var(--ml-fs-kpi-label); }
    .apriso-kpi__value { font-size: var(--ml-fs-kpi-value); }

    /* --- MACHINE GRID — assouplissement flexbox --------------------------- */

    .apriso-machine-grid {
        gap: var(--ml-spacing-3);
        padding: var(--ml-spacing-4);
    }

    .apriso-machine-card {
        /* Cible 240px, peut grandir et rétrécir, plafond raisonnable */
        flex: 1 1 var(--ml-card-width);
        max-width: var(--ml-card-max);
        width: auto;        /* annule width fixe Apriso */
        min-width: 0;       /* permet aux enfants de tronquer */

        display: flex;
        flex-direction: column;
        gap: var(--ml-spacing-2);

        background: var(--ml-color-bg-cream);
        border: 1px solid var(--ml-color-border);
        border-left: 4px solid var(--ml-color-status-idle);
        border-radius: var(--ml-radius);
        padding: var(--ml-spacing-3) var(--ml-spacing-4);

        & .apriso-machine-card__header {
            display: flex;
            align-items: center;
            gap: var(--ml-spacing-2);
            min-width: 0;       /* nécessaire pour ellipsis du titre */
        }

        & .apriso-machine-card__title {
            flex: 1 1 auto;
            min-width: 0;
            font-size: var(--ml-fs-card-title);
            color: var(--ml-color-text);
            overflow: hidden;
            text-overflow: ellipsis;
            white-space: nowrap;
        }

        & .apriso-machine-card__status {
            flex: 0 0 auto;
        }

        & .apriso-machine-card__last-event {
            display: -webkit-box;
            -webkit-line-clamp: 2;
            -webkit-box-orient: vertical;
            overflow: hidden;
        }

        &:hover { box-shadow: var(--ml-shadow-2); }

        /* Status modifiers escaladés dans @layer priority (Apriso !important) */
    }

    /* --- EVENT LOG — overflow textes longs -------------------------------- */

    .apriso-event-log {
        background: var(--ml-color-surface);
        border-left-color: var(--ml-color-border);

        & .apriso-event-log__header {
            background: var(--ml-color-bg-cream);
            border-bottom-color: var(--ml-color-border);
        }

        & .apriso-event-log__table td {
            overflow-wrap: anywhere;
        }

        /* Dernière colonne (message) — joli wrap typo */
        & .apriso-event-log__table td:last-child {
            text-wrap: pretty;
        }
    }

    /* --- FOOTER ----------------------------------------------------------- */

    .apriso-footer {
        background: var(--ml-color-bg-cream);
        border-top-color: var(--ml-color-border);
        color: var(--ml-color-text-muted);
        padding-left: max(16px, 2vw); padding-right: max(16px, 2vw);
    }
}
```

### Corrigé attendu

#### ❌ Solution

État attendu de `overrides.css` après le Module 4 (~316 lignes). Démontre Grid Layout : `grid-template-areas`, `repeat(auto-fit, minmax())` sur la grille machines, `position: sticky` sur l'en-tête du journal, `display: contents` sur le KPI board.

```css
/* ============================================================================
   FORGE × PÂTES MAMIE LULU — overrides.css
   ----------------------------------------------------------------------------
   Checkpoint après Module 4 — Grid Layout, Subgrid & layouts denses
   Cible : Edge/Chromium <= 3 ans (Baseline widely available OK)
   Last reviewed : 2026-05-18
   ----------------------------------------------------------------------------
   Ajouts vs checkpoint Module 3 :
   - Layout dashboard via grid-template-areas
   - Grille machines via repeat(auto-fit, minmax(...))
   - KPI Board <table> transformé en Grid via display: contents
   - Journal : sticky thead + scrollbar-gutter
   - Tableau dense : tabular-nums, alignement métier, zebra rows, hover
   ============================================================================
*/

@layer reset, priority, framework, overrides;

/* Apriso amené dans la cascade via une couche nommée. Le <link> direct vers
   apriso-base.css doit être retiré du HTML. Voir Module 1 §3. */
@import url("./apriso-base.css") layer(framework);

/* ----------------------------------------------------------------------------
   PRIORITY — escalade !important pour battre les 6 !important Apriso
   ---------------------------------------------------------------------------- */

@layer priority {
    .apriso-header__title { font-size: var(--ml-fs-header-title) !important; }
    .apriso-machine-card--running     { border-left-color: var(--ml-color-status-running) !important; }
    .apriso-machine-card--idle        { border-left-color: var(--ml-color-status-idle) !important; }
    .apriso-machine-card--maintenance { border-left-color: var(--ml-color-status-maintenance) !important; }
    .apriso-machine-card--alert {
        border-left-color: var(--ml-color-status-alert) !important;
        background: color-mix(in oklab, var(--ml-color-status-alert), white 90%) !important;
    }
}

/* ----------------------------------------------------------------------------
   RESET
   ---------------------------------------------------------------------------- */

@layer reset {
    :where(h1, h2, h3, h4, h5, h6, p, dl, dd, dt) {
        margin: 0;
    }

    :where(button, select, input, textarea) {
        font: inherit;
        color: inherit;
    }
}

/* ----------------------------------------------------------------------------
   OVERRIDES
   ---------------------------------------------------------------------------- */

@layer overrides {

    /* --- TOKENS ----------------------------------------------------------- */

    :root {
        /* Brand */
        --ml-color-brand-primary: #c2410c;
        --ml-color-brand-accent:  #f59e0b;
        --ml-color-bg-cream:      #fef6e4;
        --ml-color-bg-app:        #faf3e0;
        --ml-color-text:          #1f1f1f;
        --ml-color-text-muted:    #6b5d4f;
        --ml-color-border:        #e5d4b1;
        --ml-color-surface:       #ffffff;

        /* Status */
        --ml-color-status-running:     #2e7d32;
        --ml-color-status-idle:        #6c757d;
        --ml-color-status-alert:       #f57c00;
        --ml-color-status-maintenance: #1976d2;

        /* Spacing */
        --ml-spacing-1: 4px;
        --ml-spacing-2: 8px;
        --ml-spacing-3: 12px;
        --ml-spacing-4: 16px;
        --ml-spacing-5: 24px;

        /* Typo fluide */
        --ml-fs-base:        clamp(13px, 0.95vw, 15px);
        --ml-fs-header-title: clamp(16px, 1.6vw, 24px);
        --ml-fs-kpi-label:   clamp(10px, 0.9vw, 13px);
        --ml-fs-kpi-value:   clamp(20px, 2.4vw, 40px);
        --ml-fs-card-title:  clamp(13px, 1vw, 16px);

        /* Dimensions */
        --ml-dashboard-max-width: 1920px;
        --ml-card-width-min:      240px;
        --ml-sidebar-width:       380px;
        --ml-select-width:        max(180px, 14ch);

        /* Radius / shadow */
        --ml-radius-sm: 4px;
        --ml-radius:    8px;
        --ml-shadow-1:  0 1px 2px rgb(0 0 0 / 0.06);
        --ml-shadow-2:  0 4px 12px rgb(0 0 0 / 0.10);
    }

    /* --- APP -------------------------------------------------------------- */

    html {
        background: var(--ml-color-bg-app);
        color: var(--ml-color-text);
        font-size: var(--ml-fs-base);
    }

    /* --- DASHBOARD — layout via grid-template-areas ----------------------- */

    .apriso-dashboard {
        width: min(var(--ml-dashboard-max-width), 100%);
        height: 100dvh;
        background: var(--ml-color-surface);
        border-color: var(--ml-color-border);

        display: grid;
        grid-template-rows: auto auto 1fr auto;
        grid-template-columns: 1fr var(--ml-sidebar-width);
        grid-template-areas:
            "header  header"
            "kpi     kpi"
            "main    sidebar"
            "footer  footer";
    }

    .apriso-header        { grid-area: header; }
    .apriso-kpi-board     { grid-area: kpi; }
    .apriso-main          { grid-area: main; display: contents; }
    .apriso-machine-grid  { grid-area: main; }
    .apriso-event-log     { grid-area: sidebar; }
    .apriso-footer        { grid-area: footer; }

    /* --- HEADER ----------------------------------------------------------- */

    .apriso-header {
        display: flex;
        align-items: center;
        gap: var(--ml-spacing-4);
        padding-left: max(16px, 2vw); padding-right: max(16px, 2vw);
        background: var(--ml-color-brand-primary);
        border-bottom-color: color-mix(in oklab, var(--ml-color-brand-primary), black 15%);
    }

    .apriso-header__brand    { flex: 0 0 auto; }
    .apriso-header__controls { flex: 0 0 auto; }

    .apriso-header__title {
        /* font-size : escaladé dans @layer priority (Apriso !important) */
        flex: 1 1 auto;
        min-width: 0;
        text-wrap: balance;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
    }

    .apriso-line-selector__select { width: var(--ml-select-width); }

    .apriso-connection-status__dot {
        width: 0.6em;
        height: auto;
        aspect-ratio: 1;
    }

    /* --- KPI BOARD — <table> transformé en Grid --------------------------- */

    .apriso-kpi-board {
        background: var(--ml-color-bg-cream);
        border-bottom-color: var(--ml-color-border);
        padding: min(12px, 1.5vw) min(16px, 2vw);
    }

    .apriso-kpi-board__table,
    .apriso-kpi-board__table tbody {
        display: contents;
    }

    .apriso-kpi-board__table tbody tr {
        display: grid;
        grid-template-columns: repeat(4, 1fr);
        gap: var(--ml-spacing-4);
    }

    .apriso-kpi {
        border-right: 1px solid var(--ml-color-border);
    }

    .apriso-kpi:last-child { border-right: none; }

    .apriso-kpi__label { font-size: var(--ml-fs-kpi-label); }
    .apriso-kpi__value { font-size: var(--ml-fs-kpi-value); font-variant-numeric: tabular-nums; }

    /* --- MACHINE GRID — Grid avec auto-fit + minmax ----------------------- */

    .apriso-machine-grid {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(var(--ml-card-width-min), 1fr));
        gap: var(--ml-spacing-3);
        padding: var(--ml-spacing-4);
        align-content: start;
        overflow-y: auto;
        scrollbar-gutter: stable;
        background: var(--ml-color-bg-app);
    }

    .apriso-machine-card {
        /* annule width / flex hérités des modules précédents */
        width: auto;
        max-width: none;
        flex: initial;

        display: flex;
        flex-direction: column;
        gap: var(--ml-spacing-2);
        min-width: 0;

        background: var(--ml-color-bg-cream);
        border: 1px solid var(--ml-color-border);
        border-left: 4px solid var(--ml-color-status-idle);
        border-radius: var(--ml-radius);
        padding: var(--ml-spacing-3) var(--ml-spacing-4);

        & .apriso-machine-card__header {
            display: flex;
            align-items: center;
            gap: var(--ml-spacing-2);
            min-width: 0;
        }

        & .apriso-machine-card__title {
            flex: 1 1 auto;
            min-width: 0;
            font-size: var(--ml-fs-card-title);
            color: var(--ml-color-text);
            overflow: hidden;
            text-overflow: ellipsis;
            white-space: nowrap;
        }

        & .apriso-machine-card__status { flex: 0 0 auto; }

        & .apriso-machine-card__last-event {
            display: -webkit-box;
            -webkit-line-clamp: 2;
            -webkit-box-orient: vertical;
            overflow: hidden;
        }

        &:hover { box-shadow: var(--ml-shadow-2); }

        /* Status modifiers escaladés dans @layer priority (Apriso !important) */
    }

    /* --- EVENT LOG — sticky header + scrollbar-gutter + tableau dense ----- */

    .apriso-event-log {
        background: var(--ml-color-surface);
        border-left-color: var(--ml-color-border);

        & .apriso-event-log__header {
            background: var(--ml-color-bg-cream);
            border-bottom-color: var(--ml-color-border);
        }

        & .apriso-event-log__container {
            scrollbar-gutter: stable;
        }

        & .apriso-event-log__table thead th {
            position: sticky;
            top: 0;
            z-index: 1;
            background: var(--ml-color-bg-cream);
            border-bottom: 1px solid var(--ml-color-border);
        }

        /* Alignement métier */
        & .apriso-event-log__table td:first-child {        /* Heure */
            font-variant-numeric: tabular-nums;
            text-align: right;
            white-space: nowrap;
        }

        & .apriso-event-log__table td:last-child {         /* Sévérité */
            text-align: right;
            white-space: nowrap;
        }

        & .apriso-event-log__table td {
            overflow-wrap: anywhere;
        }

        /* Zebra rows — :where() pour garder la spécificité basse */
        & :where(.apriso-event-log__table tbody tr:nth-child(even)) {
            background: color-mix(in oklab, var(--ml-color-bg-cream), white 30%);
        }

        & .apriso-event-log__table tbody tr:hover {
            background: color-mix(in oklab, var(--ml-color-brand-accent), white 88%);
        }
    }

    /* --- FOOTER ----------------------------------------------------------- */

    .apriso-footer {
        background: var(--ml-color-bg-cream);
        border-top-color: var(--ml-color-border);
        color: var(--ml-color-text-muted);
        padding-left: max(16px, 2vw); padding-right: max(16px, 2vw);
    }
}
```

---

## 🚀 Pour aller plus loin

### 🎬 Démo de clôture — Subgrid (5 min, sans exercice)

Problème classique : on a une grille de cards, et **dans chaque card** on a des sous-éléments (header / metrics / footer) qu'on aimerait alignés **entre cards**.

Sans Subgrid → chaque card a sa propre mise en page interne, désalignements garantis.

Avec Subgrid :

```css
.apriso-machine-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  grid-template-rows: auto auto auto;   /* 3 zones internes */
  gap: var(--ml-spacing-3);
}

.apriso-machine-card {
  display: grid;
  grid-row: span 3;                     /* occupe les 3 lignes */
  grid-template-rows: subgrid;          /* hérite des lignes du parent */
}
```

→ Toutes les cards ont leur header / metrics / footer **alignés** entre eux, peu importe le contenu interne de chacune.

**À montrer en live, pas à faire faire.** Effet "ah ouais quand même" garanti, message à retenir : *Subgrid existe, sortez-le quand vous galérez à aligner des sous-éléments inter-composants.*

Cible Baseline : Subgrid = **Widely available** mi-2024 (Chrome/Edge 117+). Pour Dymasco : ✅ feu vert.

### Bonus 1 — `place-items` / `place-content` shorthand

```css
.apriso-kpi { place-items: center start; }   /* align-items + justify-items */
```

### Bonus 2 — Grille dense avec `grid-auto-flow: dense`

```css
.apriso-machine-grid { grid-auto-flow: dense; }
```

→ Le navigateur essaie de **combler les trous** dans la grille avec les items suivants. Utile si certaines cards sont `grid-column: span 2`.

### Bonus 3 — `min-content` / `max-content` pour les colonnes

```css
.apriso-event-log__table {
  display: grid;
  grid-template-columns: max-content max-content 1fr min-content;
}
```

→ Première colonne (heure) : largeur exacte du contenu le plus large. Dernière (sévérité) : la plus petite. Colonne message (1fr) prend le reste.

### Bonus 4 — `place-self: stretch` pour subgrid

Avec subgrid, tester `place-self` sur les enfants pour fine-tuner alignement par item.

### Ressources

- [MDN — CSS Grid Layout](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_grid_layout)
- [MDN — grid-template-areas](https://developer.mozilla.org/en-US/docs/Web/CSS/grid-template-areas)
- [MDN — Subgrid](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_grid_layout/Subgrid)
- [MDN — position: sticky](https://developer.mozilla.org/en-US/docs/Web/CSS/position#sticky_positioning)
- [MDN — scrollbar-gutter](https://developer.mozilla.org/en-US/docs/Web/CSS/scrollbar-gutter)
- [MDN — display: contents](https://developer.mozilla.org/en-US/docs/Web/CSS/display#contents)
- Rachel Andrew, [Grid by Example](https://gridbyexample.com/)

---

## 📝 Récap & checklist

- [ ] Je sais lire et écrire un layout en `grid-template-areas`
- [ ] Je connais `repeat(auto-fit, minmax(min, 1fr))` par cœur
- [ ] Je sais transformer un `<table>` en Grid via `display: contents`
- [ ] Je sais que **Subgrid existe** et sert à aligner des sous-éléments entre composants (vu en démo de clôture)
- [ ] Je connais les **4 conditions** pour que `position: sticky` fonctionne
- [ ] Je n'oublie **jamais** le `background` sur un sticky
- [ ] J'utilise `scrollbar-gutter: stable` sur tous mes conteneurs scrollables
- [ ] J'utilise `font-variant-numeric: tabular-nums` sur mes colonnes numériques

> 🎯 **Mantra du module** : *"Si je vois encore une `<table>` rigide dans un dashboard, je sais quoi faire."*
