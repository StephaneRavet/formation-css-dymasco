---
formateur:
  prerequis_formateur:
    - savoir que :has() prend la spécificité du PLUS SPÉCIFIQUE dans sa liste (idem :is())
    - connaître coût perf :has() (recalcule à chaque mutation DOM)
    - avoir testé :has() sur navigateur cible (widely 2024)
    - savoir basculer dir="rtl" dans DevTools (Elements → html → ajouter attr)
    - liste mapping physique → logique en tête (margin-left → margin-inline-start)
  ressources_demo_a_preparer:
    - checkpoint Module 4 ouvert
    - HTML avec card alerte Laminoir Léon visible
    - DevTools prêt pour basculer html lang="ar" dir="rtl"
    - tableau mapping logique imprimé
  pitch_ouverture: >
    "Levez la main si vous avez écrit du JS pour 'si la card contient un badge alert,
    colorier le parent'. C'est fini. Une ligne de CSS suffit. Bienvenue dans :has()."
  energie_attendue: haute — démarrage J2, café, effet "wow" :has() garanti
  duree_cible: 105 min — découpage 40 concepts / 20 démo / 35 exo / 10 récap
  variantes_timing:
    si_en_retard: zapper bonus :user-invalid + inset shorthand, condenser :not() en 1 min
    si_en_avance: faire chercher 3 cas :has() dans LEUR base code actuelle (mental)
  points_a_marteler:
    - ":has() = sélecteur PARENT, ENFIN possible — sans JS"
    - "spécificité :is() / :has() = celle du plus spécifique dans la liste"
    - ":where() = spécificité ZÉRO (rappel Module 1) — idéal resets/utilities"
    - "propriétés logiques = i18n RTL gratuite + writing-mode"
    - "tester :has() côté perf si DOM mute beaucoup (rare en dashboard)"
  pieges_apprenants:
    - croire :has() = ralenti TOUJOURS — non, recalcule à mutation, mesurer avant optim
    - :is(#x, .y) = spécificité 1,0,0 (id) — surprise garantie
    - mélanger physique + logique sur même élément → cascade imprévisible
    - oublier que logique dépend de writing-mode ET direction (les deux)
  questions_probables:
    - q: ":has() coûte combien en perf ?"
      r: négligeable sur 20-100 éléments. À mesurer si DOM mute 10×/sec sur 10k lignes.
    - q: "Pas de :contains() en CSS ?"
      r: non — uniquement jQuery. Passer par classe métier sur l'élément.
    - q: "RTL en industrie, on en a vraiment besoin ?"
      r: clients export Maghreb, suisse italien, futur. Coût zéro, hygiène pro.
    - q: ":is() vs :where() je prends lequel par défaut ?"
      r: :where() pour base/reset/utility. :is() quand spé assumée nécessaire.
  transition_module_suivant: >
    "Sélecteurs OK. Maintenant pivot conceptuel : nos composants doivent réagir à leur
    CONTENEUR, pas au viewport. Direction Container Queries — le Saint Graal du composant
    réutilisable, 45 minutes effet wow."
---

# Module 5 — Sélecteurs modernes & propriétés logiques

> ⏱️ **Durée** : 1h45 — **J2 matin, ouverture**
> 🧭 **Type** : module pratique (sélecteurs CSS modernes + i18n)
> 🎯 **Checkpoint** : `projet-fil-rouge/checkpoints/05-selecteurs/overrides.css`

---

## 🎯 Objectif (en 1 phrase)

Maîtriser `:is()`, `:where()`, **`:has()`** (le sélecteur "parent enfin possible") et les **propriétés logiques** (`-inline`, `-block`) pour écrire moins de code et préparer l'i18n.

---

## ⚙️ Pour qui c'est utile chez Dymasco

Trois cas concrets :

1. **Mises en évidence métier sans JS** : "si une card contient un statut critique, surligne la card entière". Avant : JS. Maintenant : `.card:has(.status--alert)`.
2. **Sélecteurs propres** sur des frameworks comme Apriso : `:is(.zone-a, .zone-b) .composant` au lieu de dupliquer.
3. **Internationalisation** : un client fictif export Maghreb (RTL arabe), ou un libellé suisse (italien). Les propriétés logiques (`margin-inline-start` au lieu de `margin-left`) **suivent automatiquement** la direction du texte.

Aujourd'hui dans `apriso-base.css` :

```css
.apriso-machine-card { border-left: 4px solid #6c757d; }   /* "left" en dur */
.apriso-event-log__table th { text-align: left; }
.apriso-machine-card { padding: 12px 14px; }   /* impossible à symétriser proprement en RTL */
```

Et aucune mise en évidence des cards en alerte (il faut chercher le badge orange à l'œil).

---

## 📚 Concepts (40 min)

### 1. `:is()` — la factorisation de sélecteurs (5 min)

```css
/* avant */
.apriso-header h1, .apriso-header h2, .apriso-header h3 { color: white; }

/* après */
.apriso-header :is(h1, h2, h3) { color: white; }
```

Spécificité : `:is()` prend la **spécificité du sélecteur le plus spécifique** dans sa liste. Important à savoir.

```css
:is(#sidebar, .sidebar) p { ... }   /* spécificité = celle de #sidebar (1,0,0) */
```

Cible Baseline : **Widely available**. ✅.

### 2. `:where()` — `:is()` sans spécificité (3 min)

Identique à `:is()`, **mais spécificité forcée à 0**. Idéal pour resets et utilities (cf. Module 1).

```css
:where(h1, h2, h3, h4, h5, h6) { margin: 0; }
```

→ N'entre jamais en conflit avec un composant. Toujours surchargeable.

### 3. `:has()` — le sélecteur parent (15 min)

**Le** sélecteur événement de CSS moderne. Permet de styler un parent **selon son enfant**, ou un élément selon **ce qu'il contient ou ce qui le suit**.

```css
/* Card qui contient un badge alerte → surligne toute la card */
.apriso-machine-card:has(.apriso-machine-card__status--alert) {
  background: color-mix(in oklab, var(--ml-color-status-alert), white 85%);
  outline: 2px solid var(--ml-color-status-alert);
}
```

→ **Sans JavaScript**. Sans dupliquer le modifier sur le parent. Sans hack.

#### Patterns puissants

```css
/* Form qui contient un champ invalide */
form:has(input:invalid) .submit { opacity: 0.5; pointer-events: none; }

/* Ligne de tableau dont la sévérité est critique */
.apriso-event:has(.apriso-event__severity--critical) td {
  background: color-mix(in oklab, var(--ml-color-status-alert), white 92%);
}

/* Section qui n'a PAS d'enfants — message vide */
.apriso-machine-grid:not(:has(.apriso-machine-card))::before {
  content: "Aucune machine sur cette ligne.";
  color: var(--ml-color-text-muted);
}

/* Élément suivi par un autre — combinateur frère */
.apriso-machine-card:has(+ .apriso-machine-card--alert) {
  margin-bottom: var(--ml-spacing-3);   /* espace avant une carte alerte */
}
```

#### Spécificité

`:has()` prend la spécificité du sélecteur **le plus spécifique** dans sa parenthèse (comme `:is()`).

#### Cible Baseline

`:has()` : **Widely available** fin 2023 / début 2024 (Chrome/Edge 105+, Safari 15.4+, Firefox 121+). Pour Dymasco : ✅. Sur cible client conservatrice → vérifier.

> ⚠️ **Coût perf** : `:has()` recalcule à chaque mutation DOM. Sur un tableau de **10 000 lignes** mis à jour 10× par seconde, ça compte. Sur un dashboard de **20 cards rafraîchi par minute** : invisible. Toujours mesurer avant d'optimiser.

### 4. `:not()` moderne (3 min)

Depuis 2021, `:not()` accepte une **liste** de sélecteurs.

```css
/* Toutes les cards sauf running et idle */
.apriso-machine-card:not(.apriso-machine-card--running, .apriso-machine-card--idle) {
  outline: 1px dashed var(--ml-color-border);
}
```

### 5. Pseudo-classes structurelles utiles (5 min)

| Sélecteur | Sens |
|---|---|
| `:nth-child(n)` | n-ième enfant (tout type) |
| `:nth-of-type(n)` | n-ième de **son type** |
| `:first-child` / `:last-child` | premier / dernier enfant |
| `:only-child` | seul enfant |
| `:empty` | pas de contenu |
| `:nth-child(odd)` / `(even)` | impairs / pairs |
| `:nth-child(2n+1 of .selector)` | filtré par sélecteur (Baseline newly available) |

```css
/* Première card de chaque ligne dans la grille */
.apriso-machine-grid > .apriso-machine-card:nth-child(4n+1) {
  /* ... */
}
```

### 6. Propriétés logiques (10 min)

Les propriétés CSS classiques sont **physiques** : `top`, `left`, `padding-left`, `margin-right`. Elles ne savent rien du sens d'écriture.

Les propriétés **logiques** suivent la direction du contenu (`ltr` vs `rtl`) :

| Physique | Logique |
|---|---|
| `margin-left` | `margin-inline-start` |
| `margin-right` | `margin-inline-end` |
| `padding-top` | `padding-block-start` |
| `padding-bottom` | `padding-block-end` |
| `border-left` | `border-inline-start` |
| `width` | `inline-size` |
| `height` | `block-size` |
| `text-align: left` | `text-align: start` |

Et les **shorthand** :

```css
.apriso-machine-card {
  margin-inline: var(--ml-spacing-2);     /* gauche+droite (en LTR) */
  padding-block: var(--ml-spacing-3);     /* haut+bas */
  inset-inline-start: 0;                  /* left en LTR / right en RTL */
}
```

#### Cas concret : la bordure colorée des cards

Avant :
```css
.apriso-machine-card { border-left: 4px solid var(--ml-color-status-idle); }
```

→ En RTL (arabe), la bordure colorée sera **à gauche** alors que le visuel attend "côté début de carte".

Après :
```css
.apriso-machine-card { border-inline-start: 4px solid var(--ml-color-status-idle); }
```

→ Bordure côté **début** de lecture, peu importe LTR ou RTL.

#### Cible Baseline

Propriétés logiques : **Widely available** depuis 2022 (`-inline`, `-block`, shorthand inclus). ✅.

---

## 🔍 Démonstration (20 min)

**Live** : ajouter une mise en évidence des cards en alerte.

### Étape 1 — Avant `:has()` (5 min)

Constat : le badge "Alerte" est petit, on le rate à l'œil quand 8 cards sont affichées. Aujourd'hui, pour résoudre, on ferait du JS qui ajoute `.card--alerted` au parent. Pénible et fragile.

### Étape 2 — Avec `:has()` (8 min)

Le HTML pour Laminoir Léon contient déjà `<span class="apriso-machine-card__status apriso-machine-card__status--alert">Alerte</span>`. Donc :

```css
@layer overrides {
  .apriso-machine-card:has(.apriso-machine-card__status--alert) {
    outline: 2px solid var(--ml-color-status-alert);
    outline-offset: -2px;
  }
}
```

→ La card alerte est **immédiatement** visible. Aucun JS, aucun modifier supplémentaire à propager.

Bonus : `:has()` sur le tableau d'événements pour surligner les lignes critiques.

> ⚠️ **Pas de `:contains()` en CSS** (uniquement jQuery / Sizzle). Pour cibler une ligne d'après son contenu textuel → on ne **peut pas** écrire `tr:has(td:contains("Critique"))`. Solution : faire poser une classe métier (`.apriso-event--critical`) côté HTML/back, puis `:has()` sur cette classe.

```css
.apriso-event--critical:has(td) td {
  background: color-mix(in oklab, var(--ml-color-status-alert), white 92%);
}
```

### Étape 3 — Switch RTL (5 min)

Dans DevTools : forcer `dir="rtl"` sur le `<html>`. Sans propriétés logiques → bordures et marges restent côté gauche, layout cassé.

Refactorer 3 propriétés en logiques (`border-inline-start`, `padding-inline`, `margin-inline-start`) → tout suit.

### Étape 4 — `:where()` reset (2 min)

Rappel Module 1 : utiliser `:where()` pour rendre les défauts de notre overrides facilement battables par les composants.

---

## 🛠️ Exercice fil rouge (35 min)

### Consigne

Reprendre le checkpoint Module 4 et :

1. **`:has()` sur cards alerte** : toute card avec un badge `--alert` reçoit un outline `--ml-color-status-alert`, fond légèrement teinté.

2. **`:has()` sur lignes critiques du journal** : toute ligne `.apriso-event--critical` reçoit un fond rouge léger, et la première colonne (heure) est mise en gras.

3. **`:has()` sur le footer** : si le `<footer>` contient un élément `.apriso-footer__item--alert` (à imaginer), changer le fond. Cas conceptuel — pas obligé d'ajouter le HTML, suffit d'écrire le CSS.

4. **Propriétés logiques — refactor systématique** :
   - `border-left-color` sur `.apriso-machine-card` → `border-inline-start-color`.
   - `margin-left: auto` sur le footer item time → `margin-inline-start: auto`.
   - Tous les `padding-left` / `padding-right` → `padding-inline`.
   - Tous les `text-align: left/right` → `text-align: start/end`.

5. **`:is()` factorisation** : trouver une chaîne dupliquée dans `overrides.css` et la factoriser via `:is()`.

6. **`:not()` liste** : appliquer un `outline: 1px dashed` aux cards qui ne sont **ni** running **ni** idle (cf. concept §4).

### Test RTL

Dans `index.html`, ajouter temporairement `<html dir="rtl">` (ou via DevTools). Vérifier que :
- La bordure colorée des cards passe à droite.
- Le sélecteur de ligne reste lisible.
- Les colonnes du journal s'inversent proprement.

### Pièges fréquents

- ❌ `:has()` avec une chaîne lourde côté gauche → recalculs perf à mesurer si DOM mute beaucoup.
- ❌ Spécificité de `:is()` / `:has()` = celle du **plus spécifique** dans la liste. `:is(#x, .y)` = spécificité d'un id.
- ❌ Mélanger propriétés physiques et logiques sur le même élément → cascade imprévisible. Choisir une convention.
- ❌ Oublier qu'une propriété logique **dépend du `writing-mode` ET du `direction`**. Tester explicitement les deux modes.

### Corrigé attendu

Voir `projet-fil-rouge/checkpoints/05-selecteurs/overrides.css`.

---

## 🚀 Pour aller plus loin

### Bonus 1 — `:has()` pour réagir au focus

```css
.apriso-machine-card:has(:focus-visible) {
  outline: 2px solid var(--ml-color-brand-primary);
}
```

→ Card qui contient un élément focus clavier → highlight global.

### Bonus 2 — `:user-invalid` (form moderne)

```css
input:user-invalid { border-color: var(--ml-color-status-alert); }
```

→ Marque erreur **uniquement après interaction** (pas dès le chargement).

### Bonus 3 — `inset` shorthand logique

```css
.tooltip {
  position: absolute;
  inset-inline: 0;        /* left + right */
  inset-block-start: 100%; /* top */
}
```

### Ressources

- [MDN — :has()](https://developer.mozilla.org/en-US/docs/Web/CSS/:has)
- [MDN — :is()](https://developer.mozilla.org/en-US/docs/Web/CSS/:is)
- [MDN — :where()](https://developer.mozilla.org/en-US/docs/Web/CSS/:where)
- [MDN — Propriétés logiques](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_logical_properties_and_values)
- Bramus, [The CSS :has() selector is way more than just a parent selector](https://www.bram.us/2021/12/21/the-css-has-selector/)

---

## 📝 Récap & checklist

- [ ] Je sais utiliser `:is()` pour factoriser
- [ ] Je sais utiliser `:where()` pour neutraliser la spécificité
- [ ] Je connais 3 cas d'usage `:has()` chez Dymasco
- [ ] Je sais que `:has()` / `:is()` prennent la spécificité du **plus spécifique** dans la liste
- [ ] Je remplace `margin-left/right` par `margin-inline-start/end` (ou `margin-inline`)
- [ ] Je remplace `text-align: left` par `text-align: start`
- [ ] Je teste mes layouts en `dir="rtl"` au moins une fois

> 🎯 **Mantra du module** : *"Avant `:has()`, on ajoutait du JS. Maintenant, on écrit une ligne de CSS."*
