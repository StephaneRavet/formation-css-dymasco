---
formateur:
  prerequis_formateur:
    - maîtriser l'algo cascade exact (origine > couche > spécificité > ordre)
    - savoir expliquer en 1 phrase pourquoi unlayered bat layered (pour normal)
    - savoir expliquer en 1 phrase pourquoi !important inverse l'ordre des couches
    - avoir testé soi-même la démo @import url() layer() la veille au soir
  ressources_demo_a_preparer:
    - projet-fil-rouge/00-base/ ouvert dans VSCode + navigateur
    - DevTools panneau Styles agrandi pour voir colonne "Layer"
    - tableau récap layered/unlayered/!important imprimé ou affiché
    - calculatrice de spécificité (specificity.keegan.st) en favori
  pitch_ouverture: >
    "Ouvrez vos derniers overrides.css. Comptez les !important. Comptez les chaînes
    > 3 classes. C'est le module qui supprime ça pour toujours."
  energie_attendue: très haute — module fondateur, 2h, c'est LE moment d'engager
  duree_cible: 120 min — découpage 40 concepts / 20 démo / 50 exo / 10 récap
  variantes_timing:
    si_en_retard: couper §1 rappel cascade (5 min) + bonus 1/2/3 + nesting passé en 2 min
    si_en_avance: ajouter live de spécificité sur leur ancien overrides.css
  points_a_marteler:
    - "@layer NE BAT PAS un framework unlayered tel quel"
    - "il faut RAMENER Apriso dans la cascade via @import url() layer()"
    - "pour les !important l'ordre des couches est INVERSÉ"
    - "priority est AVANT framework dans la déclaration (donc gagne en !important)"
    - "retirer le <link> vers apriso-base.css du HTML, sinon Apriso chargé 2 fois"
    - "Plan B si CSS Apriso injecté au runtime non-contournable : tout en !important dans priority (cf. encadré §3)"
  pieges_stagiaires:
    - croire que @layer surclasse tout, sans @import → démo étape 2 essentielle
    - oublier de retirer le <link> HTML → Apriso unlayered reprend la main
    - mettre :where() autour de la cible au lieu du scope → cible à spé 0
    - hardcoder !important "au cas où" dans overrides → règle priority cassée
  questions_probables:
    - q: "Pourquoi 4 couches et pas 3 ?"
      r: priority = pour battre !important Apriso, séparée d'overrides pour clarté.
    - q: "Et si j'ai du legacy IE ?"
      r: hors cible Dymasco (Module 0). Si vraiment, fallback hors @layer.
    - q: "Nesting natif vs Sass, je perds quelque chose ?"
      r: mixins et variables au build. Pas pour nous (pas de pipeline).
    - q: "Combien de tokens c'est trop ?"
      r: 5-10 couleurs sources + 4-6 spacings. Tout le reste dérivé.
  transition_module_suivant: >
    "Architecture posée. Maintenant on s'attaque aux pixels figés : largeurs en dur,
    typo statique. Direction module 2 — fluide partout, une seule règle pour 3 tailles
    d'écran."
---

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

Comment le navigateur tranche entre deux règles — du plus fort au plus faible :

| Rang | Mécanisme | Niveau / Score | Exemple | Bat quoi ? |
| --- | --- | --- | --- | --- |
| 1 | **`!important` user-agent** | Origine — niveau max | navigateur natif | tout |
| 2 | **`!important` CSS dev** (notre code) | Origine | `.btn { color: red !important; }` | tout CSS dev non-`!important` |
| 3 | **Style inline `style=""`** | Origine "author override" | `<div style="color:red">` | tout CSS dev non-`!important` ⚠️ pas un score `(1,0,0,0)` |
| 4 | **Couches `@layer`** (ordre décl.) | Couche déclarée **en dernier** gagne | `@layer base, theme;` → `theme` > `base` | rangs 5 + 6 |
| 5 | **Unlayered** (hors `@layer`) | Bat toute couche (non-`!important`) | `.btn { ... }` direct | toute déclaration layered |
| 6 | **Spécificité `(a, b, c)`** | a = ID, b = classe/attr/pseudo-classe, c = élément/pseudo-élément | voir sous-tableau ↓ | rang 7 à score égal |
| 7 | **Ordre d'apparition** | Dernière déclaration gagne | 2 règles identiques → la 2ᵉ | rien |

Sous-tableau spécificité (rang 6) — du plus fort au plus faible :

| Sélecteur | Score `(a, b, c)` |
| --- | --- |
| `#id` | `1, 0, 0` |
| `.a .b .c .d .e` (Apriso) | `0, 5, 0` |
| `.a .b .c` | `0, 3, 0` |
| `.card` | `0, 1, 0` |
| `div` | `0, 0, 1` |
| `*` | `0, 0, 0` |

> 📝 **Notes**
>
> - Rang 1-3 = étape "origine + importance" (tranchée avant tout calcul de score).
> - Rang 4-5 = étape "couches" (`@layer`).
> - Rang 6-7 = étape "sélecteur" (score puis ordre).
> - `!important` **inverse** l'ordre des couches : couche déclarée **en premier** gagne.
> - `style=""` et `!important` ne sont **pas** des niveaux supplémentaires de spécificité. Le tuple `(1, 0, 0, 0)` parfois vu est une notation pédagogique abusive : l'inline gagne parce qu'il est traité comme origine "author override", pas parce qu'il a un score plus élevé.

<!-- -->

> 🎯 **Le piège Apriso** : tous leurs sélecteurs sont préfixés `.apriso-dashboard .zone .composant .element` → spécificité **0, 4, 0** voire **0, 5, 0**. Pour battre ça avec la même technique, il faut **monter encore**. Mauvaise idée.

### 2. `@layer` — les couches en cascade (8 min)

`@layer` introduit un niveau d'organisation **au-dessus** de la spécificité. L'ordre des couches déclarées **prime** sur la spécificité interne à chaque couche.
### 3. `@import url() layer()` — Ramener Apriso dans la cascade (8 min)
```css
@layer framework, overrides;

@layer framework {
@layer reset {
  :where(h1, h2, h3, h4, h5, h6) { margin: 0; }
}

@layer priority {
  /* couche dédiée pour battre les !important Apriso (rare, à minimiser) */
  .apriso-machine-card--alert { background: var(--ml-color-status-alert-bg) !important; }
}

@import url("./apriso-base.css") layer(framework);

@layer overrides {
  /* sélecteurs courts, normales, battent Apriso normal */
  .apriso-machine-card { background: var(--ml-color-bg-cream); }
  .apriso-header__title { font-size: 22px; }
}
```

**Important côté HTML** : retirer le `<link rel="stylesheet" href="apriso-base.css">`. Apriso n'est plus chargé que via votre `@import layer(framework)`. Sans ça, Apriso resterait unlayered en parallèle, et reprendrait la main.

#### Pourquoi 4 couches ?

| Couche | Rôle | Position |
| --- | --- | --- |
| `reset` | resets neutres (`margin: 0`, etc.) | très basse — toujours battue |
| `priority` | escalades **!important** uniquement | **avant** `framework` → bat les !important Apriso |
| `framework` | Apriso, importé via `@import` | au milieu |
| `overrides` | vos overrides normaux (sans !important) | dernier → bat les normales Apriso |

Vérification :

- Normal : `reset < priority < framework < overrides` → **overrides normal gagne** ✓
- !important : ordre inversé → `overrides-imp < framework-imp < priority-imp` → **priority !important gagne** ✓ (et `reset` reste neutre)

#### Mantra opérationnel

- 95 % du temps : vous écrivez dans `overrides`, **sans** `!important`, sélecteurs courts.
- Quand Apriso a un `!important` (ex : `.apriso-machine-card--running { border-left-color: ... !important }`), vous escaladez dans `priority` **avec** `!important`.
- `reset` ne contient que des `:where(...)` à spécificité 0.

#### Cible Baseline

`@import` avec `layer()` : **Widely available** depuis 2022 (même date que `@layer`). ✅.

> 🚨 **Plan B — si Apriso injecte son CSS au runtime et vous n'avez pas la main sur le** `<link>`
>
> Le Plan A (ci-dessus) repose sur **retirer** le `<link rel="stylesheet" href="apriso-base.css">` du HTML rendu, pour que Apriso ne soit chargé qu'une fois via votre `@import layer(framework)`.
>
> Or **côté ScreenBuilder Apriso, ce n'est pas toujours possible** : la plateforme peut injecter ses propres feuilles de style au runtime (via `<head>` généré ASP, via JS, via une config IIS) sans exposer de point d'extension pour les retirer. À vérifier **mission par mission** : ouvrir DevTools → Network, filtrer `.css`, compter les occurrences de `apriso-base.css`. Si vous le voyez chargé en `<link>` que vous ne pouvez pas supprimer côté HTML Layout Editor → **Plan A impossible**.
> ****Conséquence** : Apriso reste **unlayered** côté navigateur, peu importe ce que vous écrivez côté overrides. La règle "unlayered = couche implicite finale = bat tous les layered normaux" s'applique → vos `@layer overrides { … }` perdent silencieusement.
> ****Plan B —** `!important` **partout dans** `@layer priority`
>
> Bonne nouvelle malgré tout : pour les `!important`, l'ordre des couches est **inversé**, et `!important` **layered bat** `!important` **unlayered**. Donc une déclaration `!important` dans n'importe quelle couche nommée bat **tout** ce que fait Apriso (normal ET `!important`).
>
> Stratégie de repli :
>
> ```css
> /* Plan B — Apriso unlayered subi, on passe TOUT en !important dans priority */
> @layer priority;
> 
> @layer priority {
>   .apriso-machine-card        { background: var(--ml-color-bg-cream) !important; }
>   .apriso-machine-card__title { color:      var(--ml-color-text)     !important; }
>   .apriso-header              { background: var(--ml-color-brand-primary) !important; }
>   /* … toute l'identité visuelle, !important obligatoire */
> }
> 
> /* Tokens dans :root — l'héritage fonctionne toujours, pas concerné par les couches */
> :root {
>   --ml-color-brand-primary: #c2410c;
>   --ml-color-bg-cream:      #fef6e4;
>   /* … */
> }
> ```
> ****Conséquences** :
>
> - **Une seule couche utile** (`priority`). Plus de distinction `overrides` cosmétique vs `priority` escalade — tout est dans la même boîte. Lisibilité réduite, mais pas catastrophique si on garde une discipline (commentaire de section par composant).
> - `@layer reset` **ne sert plus à rien** (perd contre Apriso unlayered). À supprimer du Plan B. Si reset nécessaire → l'écrire en `!important` dans `priority`, à utiliser avec parcimonie.
> - `:root` **+ tokens fonctionnent normalement** (l'héritage des custom properties n'est pas affecté par les couches — seul l'override de la **valeur** doit éventuellement passer par `!important`).
> - **Plus de "discipline** `!important`**" possible** : tout est `!important`. Difficile de signaler au lecteur "ici c'est une escalade exceptionnelle" → perdu côté maintenance.
> ****Quand choisir Plan B** :
>
> - Apriso runtime injection vérifiée non-contournable
> - OU projet pilote court (&lt; 6 mois) où la dette technique du Plan B est acceptable
> - OU politique IT cliente refuse toute modification du HTML rendu (rare mais existant)
> ****Recommandation** : tenter **toujours Plan A en premier** (test 30 min : tenter de retirer le `<link>` dans ScreenBuilder + recharger). Bascule Plan B uniquement si Plan A confirmé impossible. Documenter le choix dans le commentaire en tête d'`overrides.css` :
>
> ```css
> /* Target: <browserslist> — last reviewed: YYYY-MM-DD */
> /* Architecture: Plan B (!important dans priority) — Apriso runtime injection non-contournable, vérifié 2026-05-19 */
> ```
>
#### Anti-pattern

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

→ **Marche maintenant sans** `!important`. Apriso est dans `framework`, overrides est plus tard, overrides gagne.

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

3. **Définir le bloc** `:root` dans `@layer overrides` avec les **tokens Mamie Lulu** (cf. concept §6 ci-dessus).

4. `@layer overrides` — surcharger sans `!important`, sélecteurs courts :

   - `.apriso-header` → fond `--ml-color-brand-primary`
   - `.apriso-header__title` → `font-size: 22px` (Apriso a `!important` ici → exception, voir §5 ci-dessous)
   - `.apriso-machine-card` → fond `--ml-color-bg-cream`, bord `--ml-color-border`, radius `--ml-radius`
   - `.apriso-machine-card--running` → bord gauche `--ml-color-status-running` (Apriso a `!important` → §5)
   - `.apriso-machine-card--idle` → bord gauche `--ml-color-status-idle` (idem §5)
   - `.apriso-machine-card--alert` → bord gauche `--ml-color-status-alert` (idem §5)
   - `.apriso-machine-card--maintenance` → bord gauche `--ml-color-status-maintenance` (idem §5)

5. `@layer priority` — pour chaque rule où Apriso pose `!important`, écrire la version override **avec** `!important` ici. Ce sont les 6 cas suivants (à grep dans `apriso-base.css`) :

   - `.apriso-header__title { font-size: ... !important }`
   - `.apriso-machine-card--running { border-left-color: ... !important }`
   - `.apriso-machine-card--idle { border-left-color: ... !important }`
   - `.apriso-machine-card--alert { border-left-color: ... !important; background: ... !important }`
   - `.apriso-machine-card--maintenance { border-left-color: ... !important }`

6. **Nesting natif** dans `.apriso-machine-card`.

7. `:where()` : ajouter dans `@layer reset` un `:where(h1, h2, h3, h4) { margin: 0 }`.

### Code de départ

```css
/* projet-fil-rouge/00-base/overrides.css */
/* Vide. À toi de jouer. */
```

### Pièges fréquents (à anticiper)

- ❌ **Oublier de retirer le** `<link>` vers apriso-base.css du HTML → Apriso est chargé deux fois, dont une unlayered, qui reprend la main. **Vérifier dans DevTools → Network que apriso-base.css n'est chargé qu'une seule fois.**
- ❌ Oublier de **déclarer l'ordre** des couches en tête → ordre dépend de l'ordre d'apparition, imprévisible.
- ❌ Mettre `:root` dans `@layer reset` ou `priority` → tokens héritent partout, mais leur **valeur** doit vivre dans la couche qui les pilote (ici `overrides`).
- ❌ Imbriquer un `&:hover` sans le `&` → erreur de parsing silencieuse selon le nav.
- ❌ Utiliser `:where()` **autour du sélecteur cible** (ex : `:where(.machine-card)`) → la cible elle-même tombe à 0 spécificité. `:where()` enveloppe le **scope**, pas la cible.
- ❌ Hardcoder un `!important` "au cas où" dans `@layer overrides` → marche techniquement, mais brouille la lecture. La règle : `!important` **uniquement** dans `@layer priority`, et **uniquement** quand Apriso impose un `!important`.

### Corrigé attendu

Voir `projet-fil-rouge/checkpoints/01-architecture/overrides.css` (\~150 lignes incluant tokens et la couche `priority` pour les 6 `!important` Apriso).

---

## 🧪 Drill DevTools — Audit cascade en live (10 min, optionnel si temps)

> 🎯 **Pourquoi** : 80 % du temps quotidien chez Dymasco, vous n'écrivez pas du CSS — vous **lisez** la cascade dans DevTools pour comprendre pourquoi votre règle ne s'applique pas. Ce drill rend ce geste réflexe.

### Drill 1 — Lire le panneau Styles

1. Ouvrir le fil rouge état checkpoint M01, sélectionner `.apriso-machine-card--alert` dans Elements.
2. Dans le panneau **Styles**, repérer pour chaque déclaration :
   - le **nom de la couche** (`Layer: framework` / `Layer: overrides` / `Layer: priority`)
   - les déclarations **barrées** (overridées) avec l'icône d'info au survol
3. Question piège à se poser : *"si je vois* `priority > framework > overrides`*, qui gagne sur une déclaration normale ?"* (réponse : `overrides`, dernière couche pour les normales).

### Drill 2 — Computed + filtre

1. Onglet **Computed** → colonne **Source** : montre **le fichier + la couche** d'où vient la valeur finale.
2. Filtrer par propriété (`background`, `border-left-color`) → ne voir que les valeurs effectives.
3. Cliquer sur la valeur → DevTools déroule **toute la chaîne** des déclarations rejetées avec raison.

### Drill 3 — CSS Overview (Chrome/Edge)

`F12 → More tools → CSS Overview → Capture overview`. Affiche :

- nombre de **couleurs uniques** (avant tokens : 47, après : 9 → preuve que tokens marchent)
- nombre de **déclarations !important** (avant : 23, après pyramide : 6, **uniquement** dans `priority`)
- **contrast ratios** WCAG par paire (anticipe M10)

À faire **avant et après** votre refacto pour quantifier le gain. Capture d'écran à archiver dans le PR.

### Drill 4 — Spécificité interactive

Inspecter un sélecteur Apriso à 5 classes dans Styles → DevTools affiche le tuple `(0, 5, 0)` au survol. Comparer avec votre override `(0, 1, 0)` dans `@layer overrides` → DevTools montre **explicitement** que la spé perd, mais la **couche** gagne. Démonstration visuelle imparable du module.

---

## 🛠️ Exercice alternatif — Migration legacy "vrai monde sale" (45 min, variant pour stagiaires avancés)

> 🎯 **Pourquoi** : le fil rouge Mamie Lulu part d'un `overrides.css` vide. La réalité Dymasco = vous héritez d'un fichier de 600-1200 lignes accumulé sur 3 ans, chaînes 5 classes + `!important` partout, 0 token. Cet exo simule **ce vrai cas**.

### Code de départ : `legacy-overrides.css` (fourni)

Fichier de 500+ lignes avec les pathologies typiques :

- 47 occurrences de `!important`
- chaînes moyennes 4,2 classes
- 23 couleurs hardcodées hex/rgb (jamais de variables)
- pas de structure (composants mélangés, ordre chronologique de commits)

### Consigne — Plan en 5 étapes (à appliquer **dans cet ordre**)

1. **Auditer** (5 min) : lancer CSS Overview, archiver compteur `!important`, couleurs uniques, spé moyenne. **Point de départ chiffré**.

2. **Ajouter la pyramide sans rien casser** (5 min) :

   ```css
   @layer reset, priority, framework, overrides, legacy;
   @import url("./apriso-base.css") layer(framework);
   /* legacy/* contenu actuel reste unlayered le temps de la migration */
   ```

   Pourquoi déclarer `legacy` même si vide ? Pour avoir l'ordre verrouillé quand on commencera à y mettre des choses. La règle "unlayered = couche implicite finale" vous **protège** : votre legacy continue de marcher.

3. **Extraire les tokens** (15 min) : grep les 23 couleurs, créer `:root` dans `@layer overrides`, remplacer **uniquement** les hex/rgb dans les déclarations qui n'ont pas `!important` (les autres viendront en étape 4).

4. **Migrer composant par composant** (15 min) : pour chaque section thématique (header, cards, footer…), déplacer les règles dans `@layer overrides` (sans `!important`) et `@layer priority` (avec `!important` uniquement si le framework l'impose). Tester après chaque composant. **Jamais de big bang.**

5. **Vider** `legacy` (5 min) : à terme la couche `legacy` doit être vide et supprimée. Si elle ne l'est pas après 2 sprints → revue d'équipe, c'est probablement du JS qui pose des inline styles.

### Critères de succès

| Avant | Après cible |
| --- | --- |
| 47 `!important` | ≤ 8, **tous** dans `priority` |
| Chaînes moy 4,2 classes | ≤ 1,5 classe |
| 23 couleurs hex | 9 tokens sources + dérivés `color-mix` |
| 0 documentation cible | Header avec `browserslist` + date de revue |

### Pièges spécifiques migration

- ❌ Tout migrer en une seule PR → ingérable, code review impossible, régressions invisibles. **Une PR par composant**.
- ❌ Supprimer un `!important` legacy sans vérifier qu'il battait vraiment quelque chose → certains sont historiques, plus aucune utilité. Tester avec DevTools : désactiver `!important` à la main, observer si autre déclaration prend le dessus. Si non → `!important` mort, à virer.
- ❌ Oublier le commentaire `/* migré depuis legacy l.234 — 2026-05-18 */` dans la nouvelle règle → traçabilité perdue au prochain merge.

### Livrable

Diff git annoté + CSS Overview avant/après + commentaire de PR avec les 4 chiffres du tableau de succès. C'est le **vrai livrable Dymasco**, pas un fichier "qui rend bien".

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

### Bonus 4 — Survivre à un upgrade Apriso (defensive selectors)

Cas vécu : DELMIA Apriso upgrade `2023 → 2024`. Certains noms de classes changent (`.apriso-machine-card__title` devient `.aps-card-title`), certains IDs runtime mutent (`ctl00_xxx`), de nouveaux `!important` apparaissent. Vos overrides cassent silencieusement.

#### Stratégie

1. **Audit avant upgrade** : copier le HTML rendu Apriso v(n) avant la migration. Diff structurel vs Apriso v(n+1) → liste des classes/sélecteurs perdus.

2. **Defensive selectors** dans `overrides.css` : pour les composants critiques, **cibler par attribut data** quand possible (`[data-component="machine-card"]`), pas par classe Apriso seule.

   ```css
   /* fragile — casse à l'upgrade si classe renommée */
   @layer overrides {
     .apriso-machine-card { ... }
   }
   
   /* défensif — combine classe ET data-attribute si Apriso les expose */
   @layer overrides {
     .apriso-machine-card,
     [data-apriso-component~="machine-card"] {
       /* mêmes overrides */
     }
   }
   ```

3. **Tests visuels de régression** : Percy/Chromatic ou simple capture avant/après upgrade sur les 5 écrans les plus utilisés. Diff pixel = signal early.

4. **Couche de compat dédiée** : si un upgrade casse beaucoup, intercaler une couche `@layer compat` entre `framework` et `overrides` qui re-mappe les nouveaux noms vers vos anciens overrides :

   ```css
   @layer reset, priority, framework, compat, overrides;
   
   @layer compat {
     /* alias temporaire le temps de la migration */
     .aps-card-title { /* mêmes règles que .apriso-machine-card__title */ }
   }
   ```

   À supprimer dès que les overrides sont migrés sur les nouveaux noms.

#### Quand l'appliquer

Pas systématiquement. Coût supplémentaire \~20 % de lignes. Justifié pour :

- composants où la régression visuelle a un coût métier (badges status critiques, alertes safety)
- projets dont la durée de vie attendue dépasse une version Apriso (≥ 18 mois)

Pour des overrides cosmétiques (couleurs, espacements) : sélecteur Apriso natif suffit, on accepte de refaire au prochain upgrade.

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
- [ ] Je connais le **Plan B** si Apriso injecte son CSS au runtime sans point d'extension : tout en `!important` dans `@layer priority` (cf. encadré §3), et je le documente en tête d'`overrides.css`

> 🎯 **Mantra du module** : *"`@layer` ne bat pas un framework non layered. Il faut d'abord le ramener dans la cascade via `@import url() layer()`."*
