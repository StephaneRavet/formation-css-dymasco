# Module 1 — Architecture CSS pour overrides

> ⏱️ **Durée** : 2h00 — **J1 matin**
> 🧭 **Type** : module fondateur (cascade, spécificité, `@layer`, nesting natif, `:where()`, tokens)
> 🎯 **Checkpoint** : `projet-fil-rouge/checkpoints/01-architecture/overrides.css`

---

## 🎯 Objectif (en 1 phrase)

Reprendre la **main sur la cascade** d'un framework hostile (`apriso-base.css`) sans escalader la spécificité ni empiler des `!important`.

---

## ⚙️ Pour qui c'est utile chez Dymasco

C'est **le** module qui va définir votre rapport quotidien à Apriso. Aujourd'hui, votre `overrides.css` ressemble probablement à ça :

```css
/* avant */
.apriso-dashboard .apriso-main .apriso-machine-grid .apriso-machine-card .apriso-machine-card__title {
    color: #d32f2f !important;
}
```

→ Vous copiez la chaîne du framework, vous ajoutez `!important`, vous priez. À la prochaine version d'Apriso, ça pète.

Après ce module :

```css
/* après */
@layer overrides {
  .apriso-machine-card__title { color: var(--ml-color-danger); }
}
```

→ Sélecteur court, intention claire, palette centralisée, override **garanti** par la couche, pas par la spécificité.

---

## 📚 Concepts (40 min)

### 1. Rappel cascade & spécificité (5 min)

L'ordre dans lequel un navigateur choisit une déclaration :

1. **Origine + importance** (user-agent, user, author, `!important`)
2. **Couches** (`@layer`)
3. **Spécificité** du sélecteur (a, b, c)
4. **Ordre d'apparition** dans le code

Spécificité — rappel express :

| Sélecteur | Score (a, b, c) |
|---|---|
| `*` | 0, 0, 0 |
| `div` | 0, 0, 1 |
| `.card` | 0, 1, 0 |
| `#id` | 1, 0, 0 |
| `.a .b .c` | 0, 3, 0 |
| `.a .b .c .d .e` (Apriso) | 0, 5, 0 |
| `style=""` inline | 1, 0, 0, 0 |
| `!important` | écrase tout (à spécificité égale) |

> 🎯 **Le piège Apriso** : tous leurs sélecteurs sont préfixés `.apriso-dashboard .zone .composant .element` → spécificité **0, 4, 0** voire **0, 5, 0**. Pour battre ça avec la même technique, il faut **monter encore**. Mauvaise idée.

### 2. `@layer` — les couches en cascade (10 min)

`@layer` introduit un niveau d'organisation **au-dessus** de la spécificité. L'ordre des couches déclarées **prime** sur la spécificité interne à chaque couche.

```css
@layer reset, framework, overrides;

@layer framework {
  .apriso-dashboard .apriso-main .apriso-machine-grid .apriso-machine-card .apriso-machine-card__title {
    color: #1a1a1a; /* spécificité 0,5,0 */
  }
}

@layer overrides {
  .apriso-machine-card__title {
    color: red; /* spécificité 0,1,0 — MAIS gagne car couche après */
  }
}
```

**Règle d'or** : **dernière couche déclarée = couche gagnante**, peu importe la spécificité interne.

#### Conséquences pratiques

- On peut écrire des sélecteurs **courts** dans `overrides`, ils battent les longs du framework.
- Le code d'override redevient **lisible**.
- L'intention est explicite : "je suis dans la couche `overrides`, je sais ce que je fais".

#### Et `!important` dans tout ça ?

Surprise : **les couches inversent l'ordre de `!important`**.

- Sans `@layer` : `!important` du dernier `.css` chargé gagne.
- Avec `@layer` : `!important` de la **première** couche déclarée gagne (l'inverse !).

```css
@layer reset, framework, overrides;

@layer framework {
  .x { color: red !important; }   /* ← gagne */
}

@layer overrides {
  .x { color: blue !important; }  /* perd ! contre-intuitif */
}
```

→ Conclusion : si le framework a `!important` **dans une couche**, vous perdez. **Mais** si le framework a `!important` **hors couches** (cas Apriso), votre couche `overrides` gagne quand même, parce que l'ordre est :
**unlayered `!important` < earliest layered `!important`** … enfin ça c'est compliqué. Retenez la règle simple :

> 🟢 **Code Apriso = pas de `@layer`, donc "unlayered".** Votre couche `overrides` dépasse leurs sélecteurs **et** leurs `!important`. ✅

À tester en démo, c'est l'effet "wow" du module.

#### Cible Baseline

`@layer` : **Widely available** depuis 2022 (Chrome/Edge 99+, Firefox 97+, Safari 15.4+). Pour Dymasco : ✅ feu vert (cf. Module 0).

### 3. `:where()` — la spécificité zéro (5 min)

`:where()` enveloppe un ou plusieurs sélecteurs, et **met leur spécificité à 0**. Identique à `:is()` côté correspondance, mais sans coût de spécificité.

```css
/* spécificité = 0,1,0 (juste .apriso-machine-card) */
:where(.apriso-dashboard .apriso-main) .apriso-machine-card { ... }

/* vs sans :where() — spécificité = 0,3,0 */
.apriso-dashboard .apriso-main .apriso-machine-card { ... }
```

#### Quand l'utiliser

- Pour **désamorcer la spécificité** d'un sélecteur de portée (scope) qu'on est obligé d'écrire.
- Pour qu'un **reset** ou un **base** ne combatte jamais avec les composants.
- Pour qu'un **utility class** (`.text-center`) reste battable par un composant plus spécifique.

```css
@layer reset {
  :where(h1, h2, h3, h4) { margin: 0; }
}
/* h2 dans n'importe quel composant gagne sans effort */
```

### 4. Nesting CSS natif (5 min)

Plus besoin de Sass pour imbriquer. Depuis 2023, le nesting est **natif**.

```css
.apriso-machine-card {
  border: 1px solid var(--ml-color-border);
  padding: var(--ml-spacing-3);

  & .apriso-machine-card__title {
    font-size: var(--ml-fs-card-title);
    color: var(--ml-color-text);
  }

  &:hover {
    box-shadow: var(--ml-shadow-2);
  }

  &.apriso-machine-card--alert {
    border-left-color: var(--ml-color-status-alert);
  }
}
```

#### Règles à connaître

- Le `&` est **obligatoire** quand on combine avec une pseudo-classe ou un modifier (`&:hover`, `&.modifier`).
- Le `&` est **optionnel** pour un descendant (`& .child` ou `.child` fonctionne).
- ⚠️ **Spécificité du nesting** : un bloc imbriqué se comporte comme un sélecteur composé `:is(parent) child`. La spécificité est celle du parent **le plus spécifique** + celle de l'enfant. Ce n'est **pas** une concaténation naïve.

#### Cible Baseline

Nesting CSS natif : **Widely available** mi-2024. Pour Dymasco : ✅.

### 5. Variables CSS / tokens (10 min)

Les **custom properties** (`--ma-variable`) sont la couche **identité visuelle** de votre projet. Elles vivent à la racine, sont héritées par cascade, sont surchargeables par sélecteur.

#### Anti-pattern Apriso

```css
.apriso-machine-card--running     { border-left-color: #4caf50 !important; }
.apriso-machine-card--alert       { border-left-color: #f57c00 !important; }
.apriso-machine-card--maintenance { border-left-color: #1976d2 !important; }
```

→ 3 couleurs hardcodées, 0 sémantique, 0 réutilisation.

#### Tokens Mamie Lulu

```css
@layer overrides {
  :root {
    /* Brand */
    --ml-color-brand-primary: #c2410c;     /* tomate */
    --ml-color-brand-accent:  #f59e0b;     /* jaune œuf */
    --ml-color-bg-cream:      #fef6e4;
    --ml-color-text:          #1f1f1f;
    --ml-color-border:        #e5d4b1;

    /* Status — sémantique métier (alignée sur les modifiers Apriso) */
    --ml-color-status-running:     #2e7d32;
    --ml-color-status-idle:        #6c757d;
    --ml-color-status-alert:       #f57c00;
    --ml-color-status-maintenance: #1976d2;

    /* Spacing scale */
    --ml-spacing-1: 4px;
    --ml-spacing-2: 8px;
    --ml-spacing-3: 16px;
    --ml-spacing-4: 24px;

    /* Radius / shadow */
    --ml-radius: 8px;
    --ml-shadow-1: 0 1px 2px rgb(0 0 0 / 0.06);
    --ml-shadow-2: 0 4px 12px rgb(0 0 0 / 0.10);
  }
}
```

#### Convention de nommage (à imposer en équipe)

| Préfixe | Type | Exemple |
|---|---|---|
| `--ml-color-*` | couleurs | `--ml-color-brand-primary` |
| `--ml-spacing-*` | espacements | `--ml-spacing-3` |
| `--ml-fs-*` | font-size | `--ml-fs-card-title` |
| `--ml-radius-*` | rayons | `--ml-radius-pill` |
| `--ml-shadow-*` | ombres | `--ml-shadow-2` |

> 💡 Préfixe **projet** (`--ml-` ici pour Mamie Lulu) ou **éditeur** (`--apriso-`). Jamais de variable nue (`--primary`) → collisions garanties à terme.

### 6. Pyramide complète (5 min)

Architecture cible pour `overrides.css` :

```css
/* Target: Edge/Chromium <= 3 ans — last reviewed: 2026-05-18 */

@layer reset, framework, overrides;

@layer overrides {
  :root { /* tokens */ }

  /* Composants */
  .apriso-machine-card { ... }
  .apriso-kpi-board    { ... }
  /* etc. */
}
```

→ 3 couches déclarées, tokens en tête, sélecteurs courts, framework Apriso "unlayered" donc battu. **C'est tout.**

---

## 🔍 Démonstration (20 min)

**Live** : ouvrir `00-base/index.html` dans le navigateur, DevTools ouverts.

### Étape 1 — Tentative naïve (3 min)

Dans `overrides.css` :
```css
.apriso-machine-card__title { color: red; }
```
→ Aucun effet. DevTools montre que le sélecteur Apriso (5 classes) gagne. Spécificité 0,5,0 vs 0,1,0.

### Étape 2 — Escalade `!important` (3 min)
```css
.apriso-machine-card__title { color: red !important; }
```
→ Marche, mais introduit le piège suivant : impossible de revenir sans un `!important` encore plus fort, et plus aucune réutilisabilité.

### Étape 3 — `@layer` (5 min)

```css
@layer overrides;

@layer overrides {
  .apriso-machine-card__title { color: red; }
}
```
→ **Marche sans `!important`**. Sélecteur court. Faire admirer dans DevTools : la déclaration Apriso est barrée, la nôtre gagne via "Layer: overrides".

### Étape 4 — Bonus `!important` du framework (4 min)

Tester sur `.apriso-header__title` (`font-size: 18px !important` côté Apriso) :

```css
@layer overrides {
  .apriso-header__title { font-size: 22px; }
}
```
→ Marche **aussi sans `!important`** côté override, parce que la couche `overrides` bat les déclarations unlayered, **y compris leurs `!important`**. Effet **"wow"** garanti.

### Étape 5 — Tokens (5 min)

Remplacer la couleur en dur par une variable :
```css
@layer overrides {
  :root { --ml-color-brand-primary: #c2410c; }
  .apriso-header { background: var(--ml-color-brand-primary); }
}
```
→ Changer la valeur de `--ml-color-brand-primary` → tout suit.

---

## 🛠️ Exercice fil rouge (50 min)

### Consigne

Refactorer `projet-fil-rouge/00-base/overrides.css` (vide) pour produire l'**état attendu après Module 1** du dashboard Mamie Lulu :

1. Déclarer la pyramide `@layer reset, framework, overrides`.
2. Définir le bloc `:root` avec les **tokens Mamie Lulu** (cf. concept §5 ci-dessus).
3. Surcharger les éléments suivants **sans `!important`, avec sélecteurs courts** :
   - `.apriso-header` → fond `--ml-color-brand-primary`
   - `.apriso-header__title` → `font-size: 22px` (battre le `!important` Apriso)
   - `.apriso-machine-card` → fond `--ml-color-bg-cream`, bord `--ml-color-border`, radius `--ml-radius`
   - `.apriso-machine-card--running` → bord gauche `--ml-color-status-running`
   - `.apriso-machine-card--idle` → bord gauche `--ml-color-status-idle`
   - `.apriso-machine-card--alert` → bord gauche `--ml-color-status-alert`
   - `.apriso-machine-card--maintenance` → bord gauche `--ml-color-status-maintenance`
4. Utiliser le **nesting natif** dans le bloc `.apriso-machine-card`.
5. Utiliser `:where()` au moins une fois pour un reset (ex : `:where(h1, h2, h3, h4) { margin: 0 }` — déjà dans le base, à reprendre proprement dans la couche `reset`).

### Code de départ

```css
/* projet-fil-rouge/00-base/overrides.css */
/* Vide. À toi de jouer. */
```

### Pièges fréquents (à anticiper)

- ❌ Oublier de **déclarer l'ordre** des couches en tête → la dernière définie gagne, ordre imprévisible.
- ❌ Vouloir mettre `:root` **dans** une couche **autre** qu'`overrides` → tokens héritent partout, mais leur valeur **doit** vivre dans la couche qui les pilote (ici `overrides`).
- ❌ Imbriquer un `&:hover` sans le `&` → erreur de parsing silencieuse selon le nav.
- ❌ Utiliser `:where()` **autour du sélecteur cible** (ex : `:where(.machine-card)`) → la cible elle-même tombe à 0 spécificité, surchargée par n'importe quoi. `:where()` enveloppe le **scope**, pas la cible.
- ❌ Hardcoder un `!important` "au cas où" → couche `overrides` rend ça inutile. À chasser à la relecture.

### Corrigé attendu

Voir `projet-fil-rouge/checkpoints/01-architecture/overrides.css` (~80 lignes, lisible, zéro `!important`, zéro sélecteur > 2 classes).

---

## 🚀 Pour aller plus loin

### Bonus 1 — Couches imbriquées

```css
@layer overrides {
  @layer base, components, utilities;

  @layer components {
    .apriso-machine-card { ... }
  }

  @layer utilities {
    .text-end { text-align: end; }
  }
}
```
→ Sous-organisation interne à la couche `overrides`.

### Bonus 2 — Couche anonyme

```css
@layer { /* sans nom */
  .debug-only { outline: 2px solid magenta; }
}
```
→ Couche unique, **non-réouvrable**. Utile pour un patch ponctuel.

### Bonus 3 — `@import` dans une couche

```css
@import url("vendor/some-lib.css") layer(framework);
```
→ Importer un CSS tiers **dans** une couche, pour le neutraliser globalement. Très puissant côté intégration Apriso.

### Ressources

- [MDN — @layer](https://developer.mozilla.org/en-US/docs/Web/CSS/@layer)
- [MDN — :where()](https://developer.mozilla.org/en-US/docs/Web/CSS/:where)
- [MDN — CSS nesting](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_nesting)
- [MDN — Custom properties](https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_custom_properties)
- Bramus Van Damme, [The Future of CSS: Cascade Layers](https://www.bram.us/2021/09/15/the-future-of-css-cascade-layers-css-at-layer/)

---

## 📝 Récap & checklist

- [ ] Je sais expliquer l'**ordre de cascade** (origine, couche, spécificité, ordre)
- [ ] Je sais que **dernière couche déclarée = gagne**
- [ ] Je sais que ma couche `overrides` bat le code Apriso **unlayered**, y compris leurs `!important`
- [ ] Je n'écris **plus de `!important`** dans mes overrides
- [ ] Je n'écris **plus de chaînes > 2 classes** dans mes overrides
- [ ] J'utilise `:where()` pour les sélecteurs de portée et les resets
- [ ] J'imbrique avec le **nesting natif**, pas de Sass
- [ ] Je centralise toute couleur / espacement / rayon dans **`:root` + tokens préfixés**
- [ ] Je documente la cible navigateur en tête de fichier

> 🎯 **Mantra du module** : *"L'override propre n'écrase pas. Il prend la couche du dessus."*
