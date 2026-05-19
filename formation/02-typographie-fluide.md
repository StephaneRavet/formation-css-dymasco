---
formateur:
  prerequis_formateur:
    - lire clamp(min, ideal, max) sans hésiter
    - savoir donner 3 exemples concrets clamp / min / max sans préparer
    - connaître différence vh / dvh (bug barre adresse tablette)
    - connaître statut Baseline aspect-ratio et text-wrap (balance widely / pretty newly)
  ressources_demo_a_preparer:
    - DevTools mode responsive prêt — 3 presets 800 / 1440 / 2560 sauvegardés
    - checkpoint Module 1 ouvert (point de départ)
    - utopia.fyi en favori (générateur d'échelle typo)
  pitch_ouverture: >
    "Combien de fois vous avez livré 3 fichiers CSS — dashboard-tablette, dashboard-PC,
    dashboard-mural ? Dans 45 minutes, vous saurez n'en livrer qu'UN."
  energie_attendue: posée mais énergique — clôture J1 matin, juste avant le déjeuner
  duree_cible: 45 min — découpage 15 concepts / 10 démo / 15 exo / 5 récap
  variantes_timing:
    si_en_retard: zapper text-wrap pretty (juste citer le nom) + bonus échelle complète
    si_en_avance: faire calculer clamp() pour 3 cas à la volée avec les apprenants
  points_a_marteler:
    - "clamp() jamais sans borne min ET max — vw seul = piège mobile/mural"
    - "min() = plafonne, max() = plancher — mnémo : min RENVOIE la plus petite"
    - "aspect-ratio remplace tous les hacks padding-top: 56.25%"
    - "dvh > vh sur tablette (barre adresse mobile bugue vh)"
  pieges_apprenants:
    - mettre vw sans clamp → texte microscopique sur mobile, démesuré sur mural
    - confondre min() (plafonne) et max() (plancher) — toujours hésiter
    - cumuler aspect-ratio + width + height → conflit, l'un ignoré
    - oublier que 1vw = % viewport, PAS conteneur (anticipe Module 6)
  questions_probables:
    - q: "Pourquoi rem plutôt que px pour les font-size ?"
      r: respecte zoom utilisateur et préférence taille texte navigateur.
    - q: "text-wrap pretty c'est safe en prod ?"
      r: Newly availble — OK Dymasco, vérifier cible client si conservateur.
    - q: "calc() dans clamp(), ça marche ?"
      r: oui — clamp(14px, calc(1rem + 0.2vw), 22px) parfaitement valide.
  transition_module_suivant: >
    "Dimensions fluides : OK. Mais maintenant on a un problème nouveau — quand le
    titre d'une machine est trop long, il casse la grille. Direction module 3 :
    Flexbox avancé + dompter les textes débordants."
---

# Module 2 — Typographie & dimensions fluides

> ⏱️ **Durée** : 0h45 — **J1 matin, clôture**
> 🧭 **Type** : module court, très pratique
> 🎯 **Checkpoint** : `projet-fil-rouge/checkpoints/02-typography/overrides.css`

---

## 🎯 Objectif (en 1 phrase)

Remplacer les `px` figés par des **valeurs fluides** (`clamp()`, `min()`, `max()`) pour qu'un même dashboard tourne sur **PC d'atelier 24"**, **tablette opérateur 10"** et **écran mural 55"** sans variantes CSS.

---

## ⚙️ Pour qui c'est utile chez Dymasco

Vos dashboards ne sont pas du web public. **Pas de redimensionnement** au runtime. Mais :

- Le même module Forge est déployé sur **plusieurs types de postes** chez le même client.
- Un PC superviseur 24" et une tablette Galaxy Tab 10" ont des **largeurs très différentes**.
- Aujourd'hui : vous livrez 2 ou 3 fichiers CSS variantes ("dashboard-tablette.css", "dashboard-mural.css"). Maintenance pénible.
- Demain : **un seul fichier**, dimensions fluides, le navigateur arbitre.

L'ennemi n°1 dans `apriso-base.css` :

```css
.apriso-dashboard         { width: 1440px; }     /* fige le dashboard */
.apriso-machine-card      { width: 240px; }      /* fige les cards */
.apriso-line-selector__select { width: 220px; }  /* fige le sélecteur */
```

→ Sur tablette 800px : ascenseur horizontal. Sur mural 4K : tout collé à gauche, vide énorme.

---

## 📚 Concepts (15 min)

### 1. `clamp(min, ideal, max)` — la valeur auto-régulée

`clamp()` renvoie `ideal`, mais borne à `min` ou `max` selon le contexte.

```css
font-size: clamp(14px, 1.2vw, 22px);
```

Lecture : "minimum 14px, idéalement 1,2 % de la largeur viewport, maximum 22px".

- Tablette 800px → `1.2vw` = 9,6px → bornée à **14px**.
- PC 1440px → `1.2vw` = 17,3px → **17,3px**.
- Mural 3840px → `1.2vw` = 46px → bornée à **22px**.

Un seul code, trois résultats raisonnables.

#### Patterns utiles

| Cas | Code |
|---|---|
| KPI value qui grandit avec l'écran | `font-size: clamp(20px, 2.4vw, 40px)` |
| Padding latéral fluide | `padding: 0 clamp(12px, 2vw, 32px)` |
| Largeur de card adaptative | `width: clamp(220px, 22vw, 320px)` |

### 2. `min()` et `max()` — les bornes seules

`min()` renvoie **la plus petite** des valeurs. `max()` renvoie **la plus grande**. Très pratique pour combiner unités hétérogènes.

```css
.apriso-dashboard { width: min(1440px, 100%); }   /* plafonné à 1440px, sinon 100% du conteneur parent */
.apriso-line-selector__select { width: max(180px, 14ch); } /* au moins 14 caractères de large */
```

Règle simple :

- **`min()`** = "ne dépasse pas".
- **`max()`** = "ne descend pas en dessous".
- **`clamp()`** = les deux, plus une valeur idéale.

### 3. `calc()` — l'arithmétique CSS

```css
height: calc(100vh - 64px - 96px - 40px);   /* viewport moins header, kpi, footer */
```

Utile pour combiner unités mixtes (`vh` + `px`, `%` + `rem`).

> 💡 Combinable avec `clamp()` : `clamp(14px, calc(1rem + 0.2vw), 22px)`.

### 4. `aspect-ratio` — proportions sans hack

```css
.apriso-kpi { aspect-ratio: 4 / 1; }    /* large et plat */
.apriso-status-dot { aspect-ratio: 1; } /* carré parfait, peu importe la largeur */
```

Bye bye le `padding-top: 56.25%` et autres hacks de l'époque pré-2021.

#### Cible Baseline

`aspect-ratio` : **Widely available** depuis 2021. Pour Dymasco : ✅.

### 6. Unités à connaître pour Dymasco

| Unité | Usage typique chez Dymasco |
|---|---|
| `px` | Bordures fines, valeurs de design system fixes |
| `rem` | Espacements proportionnels au `font-size` racine |
| `ch` | Largeurs basées sur le nombre de caractères (sélecteurs, inputs) |
| `vw`/`vh` | Adaptation au format de l'écran physique |
| `%` | Parent doit avoir des valeurs fixes |
| `dvh`/`svh` | Hauteur viewport sans/avec barre nav mobile (tablette opérateur) |

> ⚠️ `vh` sur tablette = parfois bug avec la barre d'adresse. Préférer **`dvh`** (dynamic viewport height) pour la hauteur d'app plein écran.

---

## 🔍 Démonstration (10 min)

**Live** : DevTools → mode responsive, basculer entre 800px (tablette), 1440px (PC), 2560px (mural).

### Avant

```css
.apriso-dashboard { width: 1440px; }
.apriso-kpi__value { font-size: 24px; }
```

→ Tablette : scroll horizontal. Mural : valeurs KPI minuscules au centre d'un océan blanc.

### Après

```css
@layer overrides {
  .apriso-dashboard {
    width: min(1920px, 100%);   /* full sur petit écran, plafond mural */
    height: 100dvh;
  }
  .apriso-kpi__value {
    font-size: clamp(20px, 2.4vw, 40px);
  }
}
```

→ Tester les 3 largeurs en live. Effet immédiat. Pas une seule media query.

### Bonus démo : `aspect-ratio` sur status dot

Avant :

```css
.apriso-connection-status__dot { width: 8px; height: 8px; border-radius: 50%; }
```

Après :

```css
.apriso-connection-status__dot { width: 0.6em; aspect-ratio: 1; border-radius: 50%; }
```

→ Le dot grandit avec le texte parent. Plus jamais à coordonner les deux.

---

## 🛠️ Exercice fil rouge (15 min)

### 📁 Code de départ

Partez de ce fichier `overrides.css` (état après Module 1) :

- [projet-fil-rouge/checkpoints/01-architecture/overrides.css](../projet-fil-rouge/checkpoints/01-architecture/overrides.css)

### Consigne

Compléter `overrides.css` (issu du checkpoint Module 1) pour rendre le dashboard fluide :

1. **`.apriso-dashboard`** → `width: min(1920px, 100%)`, `height: 100dvh`.
2. **`.apriso-kpi__value`** → `font-size: clamp(20px, 2.4vw, 40px)`.
3. **`.apriso-kpi__label`** → `font-size: clamp(10px, 0.9vw, 13px)`.
4. **`.apriso-machine-card`** → `width: clamp(220px, 22vw, 300px)` (gardera flex pour Module 3).
5. **`.apriso-line-selector__select`** → `width: max(180px, 14ch)`.
6. **`.apriso-header__title`** → `font-size: clamp(16px, 1.6vw, 24px)`.
7. **Token typo** : ajouter `--ml-fs-kpi-value`, `--ml-fs-kpi-label` dans `:root` et les utiliser au lieu d'inliner les `clamp()` (bonus : tokens fluides).

### Pièges fréquents

- ❌ Mettre `vw` sans borne → texte minuscule en mobile, gigantesque en mural. **Toujours** borner via `clamp()`.
- ❌ Utiliser `min()` quand on voulait `max()` (et inversement). Mnémo : `min(a, b)` **renvoie la plus petite** valeur, donc plafonne.
- ❌ `aspect-ratio` + `width` + `height` simultanés → conflit, l'une des contraintes est ignorée. Choisir : `width` + `aspect-ratio`, ou `height` + `aspect-ratio`.
- ❌ Oublier que `1vw` = 1% de la **largeur du viewport**, pas du conteneur. (Pour le conteneur → Module 6, Container Queries.)

### Corrigé attendu

#### ❌ Solution

État attendu de `overrides.css` après le Module 2 (~200 lignes). Ajoute les tokens fluides `--ml-fs-*` et `--ml-card-width`, applique `clamp()` sur le header title et les KPI, `aspect-ratio: 1` sur le status dot, `100dvh` sur le dashboard.

```css
/* ============================================================================
   FORGE × PÂTES MAMIE LULU — overrides.css
   ----------------------------------------------------------------------------
   Checkpoint après Module 2 — Typographie & dimensions fluides
   ----------------------------------------------------------------------------
   Démontre, en plus du Module 1 :
   - clamp(), min(), max() sur largeurs et font-sizes
   - aspect-ratio sur status dot
   - dvh pour hauteur d'app plein écran
   - Tokens fluides --ml-fs-* (font-sizes calculées)
   ============================================================================
*/

@layer reset, priority, framework, overrides;
@import url("./apriso-base.css") layer(framework);

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

@layer reset {
    :where(h1, h2, h3, h4, h5, h6, p, dl, dd, dt) { margin: 0; }
    :where(button, select, input, textarea) { font: inherit; color: inherit; }
}

@layer overrides {

    :root {
        --ml-color-brand-primary: #c2410c;
        --ml-color-brand-accent:  #f59e0b;
        --ml-color-bg-cream:      #fef6e4;
        --ml-color-bg-app:        #faf3e0;
        --ml-color-text:          #1f1f1f;
        --ml-color-text-muted:    #6b5d4f;
        --ml-color-border:        #e5d4b1;
        --ml-color-surface:       #ffffff;

        --ml-color-status-running:     #2e7d32;
        --ml-color-status-idle:        #6c757d;
        --ml-color-status-alert:       #f57c00;
        --ml-color-status-maintenance: #1976d2;

        --ml-spacing-1: 4px;
        --ml-spacing-2: 8px;
        --ml-spacing-3: 12px;
        --ml-spacing-4: 16px;
        --ml-spacing-5: 24px;

        /* Typographie fluide */
        --ml-fs-base:        clamp(13px, 0.95vw, 15px);
        --ml-fs-header-title: clamp(16px, 1.6vw, 24px);
        --ml-fs-kpi-label:   clamp(10px, 0.9vw, 13px);
        --ml-fs-kpi-value:   clamp(20px, 2.4vw, 40px);
        --ml-fs-card-title:  clamp(13px, 1vw, 16px);

        /* Dimensions fluides */
        --ml-dashboard-max-width: 1920px;
        --ml-card-width:          clamp(220px, 22vw, 300px);
        --ml-select-width:        max(180px, 14ch);

        --ml-radius-sm: 4px;
        --ml-radius:    8px;
        --ml-shadow-1:  0 1px 2px rgb(0 0 0 / 0.06);
        --ml-shadow-2:  0 4px 12px rgb(0 0 0 / 0.10);
    }

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

    .apriso-header {
        background: var(--ml-color-brand-primary);
        border-bottom-color: color-mix(in oklab, var(--ml-color-brand-primary), black 15%);
        padding-left: max(16px, 2vw); padding-right: max(16px, 2vw);
    }

    .apriso-line-selector__select { width: var(--ml-select-width); }

    .apriso-connection-status__dot { width: 0.6em; height: auto; aspect-ratio: 1; }

    .apriso-kpi-board {
        background: var(--ml-color-bg-cream);
        border-bottom-color: var(--ml-color-border);
        padding: min(12px, 1.5vw) min(16px, 2vw);
    }

    .apriso-kpi__label { font-size: var(--ml-fs-kpi-label); }
    .apriso-kpi__value { font-size: var(--ml-fs-kpi-value); }

    .apriso-machine-card {
        width: var(--ml-card-width);
        background: var(--ml-color-bg-cream);
        border: 1px solid var(--ml-color-border);
        border-left: 4px solid var(--ml-color-status-idle);
        border-radius: var(--ml-radius);
        padding: var(--ml-spacing-3) var(--ml-spacing-4);

        & .apriso-machine-card__title { font-size: var(--ml-fs-card-title); color: var(--ml-color-text); }
        &:hover { box-shadow: var(--ml-shadow-2); }
    }

    .apriso-event-log {
        background: var(--ml-color-surface);
        border-left-color: var(--ml-color-border);
        & .apriso-event-log__header { background: var(--ml-color-bg-cream); border-bottom-color: var(--ml-color-border); }
    }

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

### Bonus 1 — Échelle typo fluide complète

Pattern "type scale" basé sur un ratio (1.2, 1.25, 1.333…) appliqué fluide :

```css
:root {
  --ml-fs-base: clamp(13px, 0.95vw, 16px);
  --ml-fs-h1:   calc(var(--ml-fs-base) * 1.8);
  --ml-fs-h2:   calc(var(--ml-fs-base) * 1.5);
  --ml-fs-h3:   calc(var(--ml-fs-base) * 1.25);
}
```

→ Toute la typo suit la base, qui suit le viewport.

### Bonus 2 — `min()` créatif pour les marges

```css
padding-left: max(16px, 5vw); padding-right: max(16px, 5vw);   /* marges qui respirent sur grand écran */
```

### Bonus 3 — `text-wrap: pretty` sur les messages d'événement

### Ressources

- [MDN — clamp()](https://developer.mozilla.org/en-US/docs/Web/CSS/clamp)
- [MDN — aspect-ratio](https://developer.mozilla.org/en-US/docs/Web/CSS/aspect-ratio)
- [Utopia.fyi](https://utopia.fyi) — générateur d'échelles typo fluides

---

## 📝 Récap & checklist

- [ ] Je sais lire `clamp(min, ideal, max)`
- [ ] Je sais quand utiliser `min()` (plafonner) vs `max()` (fixer un plancher)
- [ ] Je remplace **toute** largeur figée en `px` par une expression fluide ou `min(value, 100%)`
- [ ] Je définis les `font-size` des éléments-clés (KPI, titres) en `clamp()`
- [ ] J'utilise `aspect-ratio` au lieu des hacks `padding-top`
- [ ] J'utilise `dvh` plutôt que `vh` pour les hauteurs plein écran sur tablette

> 🎯 **Mantra du module** : *"Une seule règle CSS pour 3 tailles d'écran."*
