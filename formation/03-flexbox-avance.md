---
formateur:
  prerequis_formateur:
    - décrypter flex: <grow> <shrink> <basis> instantanément
    - savoir expliquer flex: 1 1 0 vs flex: 1 1 auto (colonnes égales vs basé contenu)
    - savoir POURQUOI min-width: 0 débloque text-overflow (default min-width: auto)
    - avoir testé que line-clamp 2 marche sur la cible navigateur Dymasco
  ressources_demo_a_preparer:
    - checkpoint Module 2 ouvert
    - HTML avec 1 machine renommée "Ligne d'extrusion sous-vide L2 — Conditionneuse Coco"
    - cheatsheet flex CSS Tricks ou Joshwcomeau ouvert en favori
  pitch_ouverture: >
    "C'est l'après-midi, on attaque le concret. Vous avez tous galéré au moins une
    fois avec text-overflow: ellipsis qui marche pas. Aujourd'hui on règle ça —
    et un secret : c'est presque JAMAIS l'élément, c'est le parent."
  energie_attendue: bonne — début d'aprem, café OK, on enchaîne
  duree_cible: 75 min — découpage 25 concepts / 15 démo / 25 exo / 10 récap
  variantes_timing:
    si_en_retard: zapper bonus pseudo ::after qui absorbe, garder triptyque overflow
    si_en_avance: faire écrire à voix haute leur réflexe ellipsis-qui-marche-pas
  points_a_marteler:
    - "flex: 1 1 240px = card qui REMPLIT l'espace (vs 0 0 240px = rigide)"
    - "min-width: 0 sur item flex = clé du text-overflow ellipsis"
    - "le piège ellipsis n'est presque JAMAIS l'élément, c'est le PARENT flex"
    - "triptyque overflow : ellipsis (1 ligne) / line-clamp (N lignes) / overflow-wrap (mot)"
    - "tester avec un nom long AVANT de livrer, toujours"
  pieges_apprenants:
    - écrire flex: 1 (raccourci ambigu = 1 1 0%) au lieu de 1 1 auto
    - oublier min-width: 0 sur les ancêtres flex de l'élément ellipsé
    - utiliser word-break: break-all → texte illisible (préférer overflow-wrap: anywhere)
    - dernier item étiré 100% avec flex-wrap → solution flex 999 1 sur ::after
  questions_probables:
    - q: "line-clamp sans -webkit- ça marche quand ?"
      r: Newly available 2024 — Dymasco OK mais préférer -webkit- pour cible mixte.
    - q: "Flexbox vs Grid pour des cards ?"
      r: Grid si alignement strict colonnes (Module 4). Flex si wrap libre.
    - q: "Pourquoi min-width: auto par défaut ?"
      r: empêche un item de devenir plus petit que son contenu (sécurité spec).
  transition_module_suivant: >
    "Flex c'est 1D. Le dashboard a aussi des layouts 2D — tableaux, KPI board, journal
    scrollable. Module 4 : Grid + transformer un <table> en grille + sticky header + Subgrid."
---

# Module 3 — Flexbox avancé

> ⏱️ **Durée** : 1h15 — **J1 après-midi, ouverture**
> 🧭 **Type** : module pratique (Flexbox + gestion textes débordants)
> 🎯 **Checkpoint** : `projet-fil-rouge/checkpoints/03-flexbox/overrides.css`

---

## 🎯 Objectif (en 1 phrase)

Maîtriser les patterns "**fixe + fluide**" en Flexbox, et **dompter les textes longs** (titres machines, libellés d'événements) sans casser la mise en page.

---

## ⚙️ Pour qui c'est utile chez Dymasco

Vos dashboards manipulent du **texte métier** que vous ne contrôlez pas :
- Noms de machines longs (*"Ligne d'extrusion sous-vide L2 — Conditionneuse Coco"*).
- Messages d'événements verbeux (*"Dérive d'épaisseur supérieure à la tolérance — référence OF-2026-0428-L2-A"*).
- Codes opérateur, références d'OF, libellés multi-langues.

Et côté layout, vous avez à composer en permanence des combinaisons **"un élément qui s'adapte + un autre qui reste à sa taille"** : header (logo fixe + titre fluide + contrôles fixes), cards qui doivent remplir l'espace dispo, etc.

Aujourd'hui, dans `apriso-base.css` :

```css
.apriso-machine-grid { display: flex; flex-wrap: wrap; }
.apriso-machine-card { flex: 0 0 auto; width: 240px; }   /* rigide */
.apriso-machine-card__title { /* aucune gestion overflow */ }
```

→ Cards qui ne remplissent pas l'espace, gros vides à droite. Et un titre long fait grandir une seule card → grille déséquilibrée.

---

## 📚 Concepts (25 min)

### 1. Rappel express (3 min)

Flexbox = **un axe principal + un axe secondaire**. Conteneur (`display: flex`) + items.

| Propriété | Sur le conteneur | Sur l'item |
|---|---|---|
| Direction | `flex-direction` | — |
| Aller à la ligne | `flex-wrap` | — |
| Espacement | `gap` | — |
| Align principal | `justify-content` | — |
| Align secondaire | `align-items` (tous) / `align-self` (un) | `align-self` |
| Taille | — | `flex` (shorthand) |

Cible Baseline : Flexbox = **Widely available** depuis ~ 2015. ✅.

### 2. La shorthand `flex` — la décrypter une fois pour toutes (8 min)

```css
flex: <flex-grow> <flex-shrink> <flex-basis>;
```

| `flex-grow` | "à quelle vitesse cet item grandit s'il reste de la place" |
| `flex-shrink` | "à quelle vitesse il rétrécit s'il manque de la place" |
| `flex-basis` | "taille idéale de départ avant ajustement" |

Patrons à mémoriser :

| Code | Sens | Cas d'usage |
|---|---|---|
| `flex: 0 0 auto` | Fixe, jamais de variation | Logo, icône d'état |
| `flex: 1 1 auto` | Partage l'espace, basé sur le contenu | Titre central qui prend ce qui reste |
| `flex: 1 1 0` | Partage **égal** entre items, ignore le contenu | Colonnes égales |
| `flex: 0 1 240px` | Largeur cible 240px, peut rétrécir, pas grandir | Card "ne dépasse pas mais s'adapte" |
| `flex: 1 1 240px` | Largeur cible 240px, peut grandir et rétrécir | **Card qui remplit l'espace** ✨ |

🎯 Pour les **cards machine** chez Mamie Lulu : `flex: 1 1 240px` est la bonne réponse. Cible 240px, mais s'étire pour combler les vides.

#### Différence cruciale `flex: 1 1 0` vs `flex: 1 1 auto`

```css
/* Trois colonnes — toutes EXACTEMENT de la même largeur */
.column { flex: 1 1 0; }

/* Trois colonnes — largeur proportionnelle à leur contenu */
.column { flex: 1 1 auto; }
```

→ `0` = on ignore le contenu. `auto` = on part du contenu et on l'ajuste.

### 3. `gap` — l'espacement enfin propre (2 min)

```css
.apriso-machine-grid { display: flex; flex-wrap: wrap; gap: 12px; }
```

Plus besoin de `margin-right` + `:last-child { margin-right: 0 }`. `gap` gère interstices internes uniquement.

Cible : **Widely available** Flex+`gap` depuis fin 2021. ✅.

### 4. Pattern "fixe + fluide + fixe" (5 min) — le header type

Cas Mamie Lulu : un header avec **logo (fixe)** + **titre (étire ce qui reste)** + **contrôles (fixes)**.

```css
.apriso-header {
  display: flex;
  align-items: center;
  gap: var(--ml-spacing-4);
}
.apriso-header__brand    { flex: 0 0 auto; }   /* fixe */
.apriso-header__title    { flex: 1 1 auto; min-width: 0; }   /* fluide */
.apriso-header__controls { flex: 0 0 auto; }   /* fixe */
```

> ⚠️ **`min-width: 0` sur l'item fluide** : par défaut, un item flex a un `min-width: auto` qui empêche le rétrécissement en dessous de la largeur du contenu. **Indispensable** pour permettre le `text-overflow: ellipsis` (cf. §5).

### 5. Textes débordants — le triptyque (7 min)

Trois techniques selon le besoin :

#### a) Une ligne, ellipsis à la fin

```css
.apriso-machine-card__title {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
```

→ "Ligne d'extrusion sous-vide…"

⚠️ Nécessite que le parent **flex** ait `min-width: 0` sinon ça ne tronque pas (l'item refuse de rétrécir sous sa largeur de contenu).

#### b) Plusieurs lignes, ellipsis à la fin

```css
.apriso-machine-card__last-event {
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}
```

→ Tronque après 2 lignes avec ellipsis. Syntaxe préfixée `-webkit-` mais reconnue partout (héritée WebKit, standardisée Baseline widely available).

> 🆕 **Standard moderne** : `line-clamp: 2` (sans `-webkit-`) arrive (Newly available 2024). Pour Dymasco aujourd'hui : utiliser le bloc préfixé, ça marche partout.

#### c) Casser un mot trop long (URL, code de référence)

```css
.apriso-event-log__table td {
  overflow-wrap: anywhere;
}
```

→ "OF-2026-0428-L2-AAAAAAAA" peut être coupé en milieu de chaîne au lieu de déborder.

| Valeur | Comportement |
|---|---|
| `overflow-wrap: normal` | Casse seulement aux espaces. Défaut. |
| `overflow-wrap: break-word` | Casse mots longs si nécessaire (legacy). |
| `overflow-wrap: anywhere` | Idem, **et** compte le mot cassé pour le calcul de largeur. **Préféré.** |
| `word-break: break-all` | Casse n'importe où. À éviter (texte illisible). |

### 6. Composer les 3 dans le même composant — Card machine type

```css
.apriso-machine-card {
  display: flex;
  flex-direction: column;
  gap: var(--ml-spacing-2);
  min-width: 0;     /* permet aux enfants de tronquer */

  & .apriso-machine-card__header {
    display: flex;
    align-items: center;
    gap: var(--ml-spacing-2);
    min-width: 0;
  }

  & .apriso-machine-card__title {
    flex: 1 1 auto;
    min-width: 0;
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
}
```

→ Header de card : status badge fixe + titre fluide ellipsé. Bas de card : message dernier événement clampé à 2 lignes.

---

## 🔍 Démonstration (15 min)

**Live** : DevTools, modifier les noms de machine en direct pour voir le comportement.

### Étape 1 — Le piège initial (3 min)

`.apriso-machine-grid` avec `.apriso-machine-card { flex: 0 0 auto; width: 240px }`. Sur un écran de 1600px, on voit le grand vide à droite. Resize à 1100px → déjà mieux mais cards toujours figées.

### Étape 2 — `flex: 1 1 240px` (4 min)

```css
@layer overrides {
  .apriso-machine-card {
    flex: 1 1 240px;
    width: auto;          /* annule le width fixe du framework */
  }
}
```

→ Cards remplissent toute la ligne. Resize → cards rétrécissent jusqu'à wrap.

### Étape 3 — Casser le titre (4 min)

Renommer une machine "Ligne d'extrusion sous-vide L2 — Conditionneuse Coco" dans le HTML.

Sans gestion : la card grandit, la grille se décale.

Ajouter :
```css
.apriso-machine-card__title {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  min-width: 0;
}
```

→ Échec. **Pourquoi ?** Parent flex sans `min-width: 0`. Ajouter sur `.apriso-machine-card__header` → ça marche.

Effet pédagogique : le piège "l'ellipsis ne marche pas" n'est presque jamais le child, c'est le **parent flex**.

### Étape 4 — Clamp 2 lignes sur message événement (4 min)

```css
.apriso-machine-card__last-event {
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}
```

Tester avec un message long. → Trois petits points en fin de 2e ligne.

---

## 🛠️ Exercice fil rouge (25 min)

### Consigne

Reprendre le checkpoint Module 2 et :

1. **Header** — passer en pattern `fixe + fluide + fixe` :
   - `.apriso-header` reçoit `display: flex`, `align-items: center`, `gap`.
   - `.apriso-header__brand` → `flex: 0 0 auto`.
   - `.apriso-header__title` → `flex: 1 1 auto`, `min-width: 0`, ellipsis si trop long.
   - `.apriso-header__controls` → `flex: 0 0 auto`.

2. **Grille machines** — assouplir :
   - `.apriso-machine-grid` reste flex-wrap, `gap` token.
   - `.apriso-machine-card` → `flex: 1 1 var(--ml-card-width)` au lieu de `width: var(--ml-card-width)`. Garder un plafond raisonnable (`max-width: 360px` par exemple).

3. **Card** — composition interne :
   - `.apriso-machine-card` devient un flex-column avec gap.
   - `.apriso-machine-card__header` flex-row, min-width 0.
   - `.apriso-machine-card__title` ellipsis 1 ligne.
   - `.apriso-machine-card__status` flex 0 0 auto.
   - `.apriso-machine-card__last-event` line-clamp 2.

4. **Journal d'événements** — débordement :
   - `.apriso-event-log__table td` → `overflow-wrap: anywhere`.
   - Sur la dernière colonne (message), `text-wrap: pretty` (rappel module 2) en bonus.

5. **Test à faire** : éditer le HTML, donner à 2 machines un nom **très long**, recharger. Vérifier que :
   - Le titre s'ellipse, ne fait pas grandir la card.
   - La hauteur des cards reste alignée dans la grille.
   - Le message d'événement long se clampe à 2 lignes.

### Code de départ

Le `overrides.css` issu du checkpoint Module 2.

### Pièges fréquents

- ❌ `text-overflow: ellipsis` qui ne marche pas → vérifier `min-width: 0` sur **tous les ancêtres flex** jusqu'à l'élément.
- ❌ Mettre `flex: 1` (raccourci ambigu) au lieu de `flex: 1 1 auto` ou `flex: 1 1 0`. `flex: 1` = `1 1 0%` → ignore le contenu, pas toujours ce qu'on veut.
- ❌ Utiliser `word-break: break-all` partout → texte illisible. Préférer `overflow-wrap: anywhere`.
- ❌ Oublier que `flex-wrap: wrap` redistribue les items à chaque ligne → le dernier élément peut être tout seul, étiré à 100% si `flex-grow: 1`. Solution : `max-width` sur l'item, ou `flex-basis` réfléchi.

### Corrigé attendu

Voir `projet-fil-rouge/checkpoints/03-flexbox/overrides.css`.

---

## 🚀 Pour aller plus loin

### Bonus 1 — `flex-basis: 0` vs `auto`

```css
/* 4 colonnes égales, peu importe leur contenu */
.column { flex: 1 1 0; }

/* 4 colonnes proportionnelles à leur contenu */
.column { flex: 1 1 auto; }
```

Pratique pour comprendre ce qu'on demande au moteur de mise en page.

### Bonus 2 — `align-self` pour casser l'alignement d'un seul item

```css
.apriso-machine-card__status { align-self: flex-start; }
```

→ Le badge status reste en haut quand la card grandit, le reste s'aligne au centre.

### Bonus 3 — Empêcher le `flex-grow` final disgracieux

```css
.apriso-machine-grid::after {
  content: "";
  flex: 999 1 var(--ml-card-width);
}
```

→ Pseudo-élément invisible qui absorbe l'espace résiduel sur la dernière ligne. Évite l'effet "dernière card étirée à 100%".

### Ressources

- [MDN — Flexbox](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_flexible_box_layout)
- [MDN — flex shorthand](https://developer.mozilla.org/en-US/docs/Web/CSS/flex)
- [MDN — text-overflow](https://developer.mozilla.org/en-US/docs/Web/CSS/text-overflow)
- [MDN — overflow-wrap](https://developer.mozilla.org/en-US/docs/Web/CSS/overflow-wrap)
- [MDN — line-clamp](https://developer.mozilla.org/en-US/docs/Web/CSS/-webkit-line-clamp)
- Josh W. Comeau, [Interactive guide to Flexbox](https://www.joshwcomeau.com/css/interactive-guide-to-flexbox/)

---

## 📝 Récap & checklist

- [ ] Je décrypte `flex: <grow> <shrink> <basis>` sans hésiter
- [ ] Je distingue `flex: 1 1 0` (colonnes égales) de `flex: 1 1 auto` (basé contenu)
- [ ] J'utilise `flex: 1 1 240px` pour les cards qui doivent **remplir l'espace**
- [ ] Je pose `min-width: 0` sur les items flex qui doivent pouvoir rétrécir
- [ ] J'utilise `gap` à la place des marges entre enfants
- [ ] Je connais le triptyque overflow : `text-overflow: ellipsis`, `-webkit-line-clamp`, `overflow-wrap: anywhere`
- [ ] Je teste mes layouts avec **du texte long** avant de livrer

> 🎯 **Mantra du module** : *"Si l'ellipsis ne marche pas, c'est le parent."*
