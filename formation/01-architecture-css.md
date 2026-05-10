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
/* après — overrides.css */
@layer reset, priority, framework, overrides;
@import url("./apriso-base.css") layer(framework);

@layer overrides {
  .apriso-machine-card__title { color: var(--ml-color-danger); }
}
```

→ Sélecteur court, intention claire, palette centralisée, override **garanti** par la couche, pas par la spécificité. Et le tour de force : ramener Apriso **dans** la cascade en l'important dans une couche nommée.

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

### 2. `@layer` — les couches en cascade (8 min)

`@layer` introduit un niveau d'organisation **au-dessus** de la spécificité. L'ordre des couches déclarées **prime** sur la spécificité interne à chaque couche.

```css
@layer framework, overrides;

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

**Règle d'or — déclarations normales** : **dernière couche déclarée = couche gagnante**, peu importe la spécificité interne.

#### Et les déclarations unlayered ?

Question piège, et réponse contre-intuitive : **les déclarations sans `@layer` (unlayered) sont placées dans une "couche implicite finale"**. Pour les déclarations normales, dernière couche = gagne → **les déclarations unlayered battent toutes les déclarations layered**.

```css
@layer overrides {
  .x { color: blue; }   /* layered → perd */
}

.x { color: red; }      /* unlayered → "dernière couche" → gagne ❌ */
```

> 🚨 **Conséquence directe pour Dymasco** : `apriso-base.css` est chargé **sans `@layer`**. Si vous écrivez juste `@layer overrides { .x { ... } }` dans votre CSS, vous **perdez** contre Apriso (normal et !important).
>
> Pour reprendre la main, il faut **amener Apriso dans une couche nommée** (cf. §3 ci-dessous).

#### Et `!important` dans tout ça ?

L'ordre est **inversé** pour les `!important` : entre couches nommées, c'est la **première déclarée** qui gagne. Et la "couche implicite finale" devient la moins prioritaire. Tableau récapitulatif :

| Déclaration A | Déclaration B | Qui gagne ? |
|---|---|---|
| Normal layered (early) | Normal layered (late) | **B** (later wins) |
| Normal layered | Normal unlayered | **B** (unlayered wins) |
| Normal | !important | **!important** |
| !important layered (early) | !important layered (late) | **A** (earlier wins) |
| !important layered | !important unlayered | **layered** (unlayered loses) |

Lecture utile pour la suite :
- **Pour battre du normal Apriso unlayered** → il faut amener Apriso dans une couche nommée (§3).
- **Pour battre du !important Apriso unlayered** → suffit d'un `!important` dans une couche nommée (n'importe laquelle).

#### Cible Baseline

`@layer` : **Widely available** depuis 2022 (Chrome/Edge 99+, Firefox 97+, Safari 15.4+). Pour Dymasco : ✅ feu vert (cf. Module 0).

### 3. `@import url() layer()` — amener Apriso dans la cascade (8 min)

C'est **le** geste qui débloque tout. On charge `apriso-base.css` **dans** une couche nommée, via `@import` côté override :

```css
/* overrides.css — premier chargé par le HTML */
@layer reset, priority, framework, overrides;

@import url("./apriso-base.css") layer(framework);

@layer reset {
  :where(h1, h2, h3, h4, h5, h6) { margin: 0; }
}

@layer overrides {
  /* sélecteurs courts, normales, battent Apriso normal */
  .apriso-machine-card { background: var(--ml-color-bg-cream); }
  .apriso-header__title { font-size: 22px; }
}

@layer priority {
  /* couche dédiée pour battre les !important Apriso (rare, à minimiser) */
  .apriso-machine-card--alert { background: var(--ml-color-status-alert-bg) !important; }
}
```

**Important côté HTML** : retirer le `<link rel="stylesheet" href="apriso-base.css">`. Apriso n'est plus chargé que via votre `@import layer(framework)`. Sans ça, Apriso resterait unlayered en parallèle, et reprendrait la main.

#### Pourquoi 4 couches ?

| Couche | Rôle | Position |
|---|---|---|
| `reset` | resets neutres (`margin: 0`, etc.) | très basse — toujours battue |
| `priority` | escalades **!important** uniquement | **avant** `framework` → bat les !important Apriso |
| `framework` | Apriso, importé via `@import` | au milieu |
| `overrides` | vos overrides normaux (sans !important) | dernier → bat les normales Apriso |

Vérification :
- Normal : `reset < priority < framework < overrides` → **overrides normal gagne** ✓
- !important : ordre inversé → `overrides-imp < framework-imp < priority-imp` → **priority !important gagne** ✓ (et `reset` reste neutre)

#### Mantra opérationnel

- 95 % du temps : vous écrivez dans `overrides`, **sans `!important`**, sélecteurs courts.
- Quand Apriso a un `!important` (ex : `.apriso-machine-card--running { border-left-color: ... !important }`), vous escaladez dans `priority` **avec** `!important`.
- `reset` ne contient que des `:where(...)` à spécificité 0.

#### Cible Baseline

`@import` avec `layer()` : **Widely available** depuis 2022 (même date que `@layer`). ✅.

### 4. `:where()` — la spécificité zéro (5 min)

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

### 5. Nesting CSS natif (5 min)

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

### 6. Variables CSS / tokens (8 min)

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

### 7. Pyramide complète (4 min)

Architecture cible pour `overrides.css` :

```css
/* Target: Edge/Chromium <= 3 ans — last reviewed: 2026-05-18 */

@layer reset, priority, framework, overrides;

@import url("./apriso-base.css") layer(framework);

@layer reset {
  :where(h1, h2, h3, h4) { margin: 0; }
}

@layer overrides {
  :root { /* tokens */ }

  /* Composants — sélecteurs courts, normales */
  .apriso-machine-card { ... }
  .apriso-kpi-board    { ... }
}

@layer priority {
  /* Quand Apriso a !important : escalader ici, avec !important */
  .apriso-machine-card--alert { background: var(--ml-color-status-alert-bg) !important; }
}
```

→ **4 couches** : `reset` (très basse), `priority` (pour battre les !important), `framework` (Apriso wrappé), `overrides` (vos overrides normaux). Coût HTML : retirer le `<link>` vers `apriso-base.css`.

---

## 🔍 Démonstration (20 min)

**Live** : ouvrir `00-base/index.html` dans le navigateur, DevTools ouverts.

### Étape 1 — Tentative naïve (3 min)

Dans `overrides.css` :
```css
.apriso-machine-card__title { color: red; }
```
→ Aucun effet. DevTools montre que le sélecteur Apriso (5 classes) gagne. Spécificité 0,5,0 vs 0,1,0.

### Étape 2 — `@layer` "tel quel", le piège (4 min)

```css
@layer overrides;

@layer overrides {
  .apriso-machine-card__title { color: red; }
}
```
→ **N'a aucun effet non plus**. Apriso est unlayered → couche implicite finale → bat la couche `overrides`.

DevTools, panneau "Styles" : la règle est **présente** mais barrée par la déclaration Apriso. Lecture : `Layer: <implicit outer layer> > Layer: overrides`.

C'est le moment pédagogique critique : démystifier la croyance "@layer surclasse tout".

### Étape 3 — `@import layer(framework)` — la vraie solution (5 min)

1. Retirer du HTML le `<link rel="stylesheet" href="apriso-base.css">`.
2. Dans `overrides.css` :
```css
@layer reset, priority, framework, overrides;

@import url("./apriso-base.css") layer(framework);

@layer overrides {
  .apriso-machine-card__title { color: red; }
}
```
→ **Marche maintenant sans `!important`**. Apriso est dans `framework`, overrides est plus tard, overrides gagne.

DevTools : `Layer: framework > Layer: overrides`, la déclaration Apriso est barrée comme attendu. **Effet "wow" légitime.**

### Étape 4 — Battre les `!important` Apriso (4 min)

Tester sur `.apriso-machine-card--alert` (Apriso : `background: #fff8e1 !important`) :

```css
@layer overrides {
  .apriso-machine-card--alert { background: var(--ml-color-status-alert-bg); }
}
```
→ **Échec.** Le `!important` framework gagne contre une normale overrides.

Solution : escalader dans la couche `priority` **avec** `!important` :
```css
@layer priority {
  .apriso-machine-card--alert { background: var(--ml-color-status-alert-bg) !important; }
}
```
→ Marche. `priority` est déclarée AVANT `framework` → pour les `!important`, ordre inversé → priority gagne.

### Étape 5 — Tokens (3 min)

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

1. **Modifier le HTML** : retirer `<link rel="stylesheet" href="apriso-base.css">`. Apriso sera chargé via `@import` dans overrides.css.

2. **Déclarer la pyramide** en tête : `@layer reset, priority, framework, overrides;` puis `@import url("./apriso-base.css") layer(framework);`.

3. **Définir le bloc `:root`** dans `@layer overrides` avec les **tokens Mamie Lulu** (cf. concept §6 ci-dessus).

4. **`@layer overrides`** — surcharger sans `!important`, sélecteurs courts :
   - `.apriso-header` → fond `--ml-color-brand-primary`
   - `.apriso-header__title` → `font-size: 22px` (Apriso a `!important` ici → exception, voir §5 ci-dessous)
   - `.apriso-machine-card` → fond `--ml-color-bg-cream`, bord `--ml-color-border`, radius `--ml-radius`
   - `.apriso-machine-card--running` → bord gauche `--ml-color-status-running` (Apriso a `!important` → §5)
   - `.apriso-machine-card--idle` → bord gauche `--ml-color-status-idle` (idem §5)
   - `.apriso-machine-card--alert` → bord gauche `--ml-color-status-alert` (idem §5)
   - `.apriso-machine-card--maintenance` → bord gauche `--ml-color-status-maintenance` (idem §5)

5. **`@layer priority`** — pour chaque rule où Apriso pose `!important`, écrire la version override **avec** `!important` ici. Ce sont les 6 cas suivants (à grep dans `apriso-base.css`) :
   - `.apriso-header__title { font-size: ... !important }`
   - `.apriso-machine-card--running { border-left-color: ... !important }`
   - `.apriso-machine-card--idle { border-left-color: ... !important }`
   - `.apriso-machine-card--alert { border-left-color: ... !important; background: ... !important }`
   - `.apriso-machine-card--maintenance { border-left-color: ... !important }`

6. **Nesting natif** dans `.apriso-machine-card`.

7. **`:where()`** : ajouter dans `@layer reset` un `:where(h1, h2, h3, h4) { margin: 0 }`.

### Code de départ

```css
/* projet-fil-rouge/00-base/overrides.css */
/* Vide. À toi de jouer. */
```

### Pièges fréquents (à anticiper)

- ❌ **Oublier de retirer le `<link>`** vers apriso-base.css du HTML → Apriso est chargé deux fois, dont une unlayered, qui reprend la main. **Vérifier dans DevTools → Network que apriso-base.css n'est chargé qu'une seule fois.**
- ❌ Oublier de **déclarer l'ordre** des couches en tête → ordre dépend de l'ordre d'apparition, imprévisible.
- ❌ Mettre `:root` dans `@layer reset` ou `priority` → tokens héritent partout, mais leur **valeur** doit vivre dans la couche qui les pilote (ici `overrides`).
- ❌ Imbriquer un `&:hover` sans le `&` → erreur de parsing silencieuse selon le nav.
- ❌ Utiliser `:where()` **autour du sélecteur cible** (ex : `:where(.machine-card)`) → la cible elle-même tombe à 0 spécificité. `:where()` enveloppe le **scope**, pas la cible.
- ❌ Hardcoder un `!important` "au cas où" dans `@layer overrides` → marche techniquement, mais brouille la lecture. La règle : `!important` **uniquement** dans `@layer priority`, et **uniquement** quand Apriso impose un `!important`.

### Corrigé attendu

Voir `projet-fil-rouge/checkpoints/01-architecture/overrides.css` (~150 lignes incluant tokens et la couche `priority` pour les 6 `!important` Apriso).

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
- [ ] Je sais que pour les **normales** : dernière couche déclarée = gagne, et **unlayered = couche implicite finale = bat tous les layered**
- [ ] Je sais que pour les **!important** : ordre **inversé**, première couche déclarée = gagne, et **unlayered = perd contre tous les layered**
- [ ] Je sais ramener un framework non layered dans la cascade via `@import url() layer(name)`
- [ ] Je connais la pyramide `reset, priority, framework, overrides`
- [ ] Je n'écris `!important` que dans `priority`, et seulement quand le framework m'y force
- [ ] Je n'écris plus de chaînes > 2 classes dans mes overrides
- [ ] J'utilise `:where()` pour les sélecteurs de portée et les resets
- [ ] J'imbrique avec le **nesting natif**, pas de Sass
- [ ] Je centralise toute couleur / espacement / rayon dans **`:root` + tokens préfixés**
- [ ] Je documente la cible navigateur en tête de fichier
- [ ] Je vérifie dans **DevTools → Network** que `apriso-base.css` n'est chargé qu'**une seule fois** (via `@import`, pas via `<link>`)

> 🎯 **Mantra du module** : *"`@layer` ne bat pas un framework non layered. Il faut d'abord le ramener dans la cascade via `@import url() layer()`."*
