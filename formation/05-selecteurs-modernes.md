---
formateur:
  prerequis_formateur:
    - savoir que :has() prend la spécificité du PLUS SPÉCIFIQUE dans sa liste (idem :is())
    - connaître coût perf :has() (recalcule à chaque mutation DOM)
    - avoir testé :has() sur navigateur cible (widely 2024)
> ⏱️ **Durée** : 1h45 — **J2 matin, ouverture**
>
> 🧭 **Type** : module pratique (sélecteurs CSS modernes + propriétés logiques)
>
> 🎯 **Checkpoint** : `projet-fil-rouge/checkpoints/05-selecteurs/overrides.css`
    - checkpoint Module 4 ouvert
    - HTML avec card alerte Laminoir Léon visible
  pitch_ouverture: >
    "Levez la main si vous avez écrit du JS pour 'si la card contient un badge alert,
    colorier le parent'. C'est fini. Une ligne de CSS suffit. Bienvenue dans :has()."
  energie_attendue: haute — démarrage J2, café, effet "wow" :has() garanti
  duree_cible: 105 min — découpage 40 concepts / 20 démo / 35 exo / 10 récap
  variantes_timing:
    si_en_retard: zapper bonus :user-invalid, condenser :not() en 1 min
    si_en_avance: faire chercher 3 cas :has() dans LEUR base code actuelle (mental)
  points_a_marteler:
    - ":has() = sélecteur PARENT, ENFIN possible — sans JS"
    - "spécificité :is() / :has() = celle du plus spécifique dans la liste"
    - ":where() = spécificité ZÉRO (rappel Module 1) — idéal resets/utilities"
    - "tester :has() côté perf si DOM mute beaucoup (rare en dashboard)"
  pieges_apprenants:
    - croire :has() = ralenti TOUJOURS — non, recalcule à mutation, mesurer avant optim
    - :is(#x, .y) = spécificité 1,0,0 (id) — surprise garantie
  questions_probables:
    - q: ":has() coûte combien en perf ?"
      r: négligeable sur 20-100 éléments. À mesurer si DOM mute 10×/sec sur 10k lignes.
    - q: "Pas de :contains() en CSS ?"
      r: non — uniquement jQuery. Passer par classe métier sur l'élément.
    - q: ":is() vs :where() je prends lequel par défaut ?"
      r: :where() pour base/reset/utility. :is() quand spé assumée nécessaire.
  transition_module_suivant: >
    "Sélecteurs OK. Maintenant pivot conceptuel : nos composants doivent réagir à leur
    CONTENEUR, pas au viewport. Direction Container Queries — le Saint Graal du composant
    réutilisable, 45 minutes effet wow."
---

# Module 5 — Sélecteurs modernes

> ⏱️ **Durée** : 1h45 — **J2 matin, ouverture**
> 🧭 **Type** : module pratique (sélecteurs CSS modernes)
> 🎯 **Checkpoint** : `projet-fil-rouge/checkpoints/05-selecteurs/overrides.css`

---

## 🎯 Objectif (en 1 phrase)

> 💡 Tous ces combinateurs fonctionnent **à l'intérieur** des pseudo-classes fontionnelles `:is()`, `:where()`, `:not()` et `:has()`.

---

## ⚙️ Pour qui c'est utile chez Dymasco

Deux cas concrets :

1. **Mises en évidence métier sans JS** : "si une card contient un statut critique, surligne la card entière". Avant : JS. Maintenant : `.card:has(.status--alert)`.
2. **Sélecteurs propres** sur des frameworks comme Apriso : `:is(.zone-a, .zone-b) .composant` au lieu de dupliquer.

Aujourd'hui, aucune mise en évidence des cards en alerte (il faut chercher le badge orange à l'œil).

---

## 📚 Concepts (40 min)

### 0. Rappel — combinateurs CSS

Avant d'entrer dans `:is()` / `:has()`, les **combinateurs** de base qui reviennent partout :

| Combinateur | Nom | Exemple | Sens |
|---|---|---|---|
| (espace) | Descendant | `.card p` | un `p` **n'importe où** sous `.card` |
| `>` | Enfant direct | `.card > p` | un `p` qui est **enfant immédiat** de `.card` |
| `+` | Frère adjacent | `h2 + p` | le `p` qui suit **immédiatement** un `h2` (même parent, juste après) |
| `~` | Frère général | `h2 ~ p` | tout `p` qui suit un `h2` au même niveau (pas forcément collé) |
| `,` | Liste de sélecteurs | `h1, h2, h3` | l'un **ou** l'autre — applique à chacun |
| `&` | Référence parent (nesting) | `.card { &:hover { … } }` | équivalent imbriqué de `.card:hover` (Baseline 2023) |

> 💡 Tous ces combinateurs fonctionnent **à l'intérieur** de `:is()`, `:where()`, `:not()` et `:has()`.

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
```

> 📌 **`input:invalid` — la condition d'erreur**
> Le navigateur possède un **moteur de validation HTML5 natif** (activé dès que tu utilises des attributs comme `required`, `type="email"`, `pattern="..."`, etc.).
>
> Dès qu'un champ est vide alors qu'il est obligatoire, ou mal rempli (ex : un email sans le `@`), le navigateur lui attribue automatiquement l'état `:invalid`. Aucun JS requis — `:has(input:invalid)` réagit à cet état dès le rendu.

```css
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
| `:nth-of-type(n)` | n-ième de **son type** — ex : `article p:nth-of-type(2)` = le 2ᵉ `<p>` de l'`<article>`, en ignorant les `<h2>` ou `<ul>` intercalés (`:nth-child(2)` aurait visé le 2ᵉ enfant tout type confondu) |
| `:first-child` / `:last-child` | premier / dernier enfant |
| `:only-child` | seul enfant |
| `:empty` | pas de contenu |
| `:nth-child(odd)` / `(even)` | impairs / pairs |
| `:nth-child(2n+1 of .selector)` | filtré par sélecteur (Baseline newly available) — ex : `tr:nth-child(odd of .apriso-event--critical)` = 1 ligne sur 2 **parmi les lignes critiques uniquement** (le compteur ignore les autres `tr`, contrairement à `tr.apriso-event--critical:nth-child(odd)` qui compte sur l'ensemble) |

```css
/* Première card de chaque ligne dans la grille */
.apriso-machine-grid > .apriso-machine-card:nth-child(4n+1) {
  /* ... */
}
```

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

### Étape 3 — `:where()` reset (2 min)

Rappel Module 1 : utiliser `:where()` pour rendre les défauts de notre overrides facilement battables par les composants.

---

## 🛠️ Exercice fil rouge (35 min)

### Consigne

Reprendre le checkpoint Module 4 et :

1. **`:has()` sur cards alerte** : toute card avec un badge `--alert` reçoit un outline `--ml-color-status-alert`, fond légèrement teinté.

2. **`:has()` sur lignes critiques du journal** : toute ligne `.apriso-event--critical` reçoit un fond rouge léger, et la première colonne (heure) est mise en gras.

3. **`:has()` sur le footer** : si le `<footer>` contient un élément `.apriso-footer__item--alert` (à imaginer), changer le fond. Cas conceptuel — pas obligé d'ajouter le HTML, suffit d'écrire le CSS.

4. **`:is()` factorisation** : trouver une chaîne dupliquée dans `overrides.css` et la factoriser via `:is()`.

5. **`:not()` liste** : appliquer un `outline: 1px dashed` aux cards qui ne sont **ni** running **ni** idle.

### Pièges fréquents

- ❌ `:has()` avec une chaîne lourde côté gauche → recalculs perf à mesurer si DOM mute beaucoup.
- ❌ Spécificité de `:is()` / `:has()` = celle du **plus spécifique** dans la liste. `:is(#x, .y)` = spécificité d'un id.

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

### Ressources

- [MDN — :has()](https://developer.mozilla.org/en-US/docs/Web/CSS/:has)
- [MDN — :is()](https://developer.mozilla.org/en-US/docs/Web/CSS/:is)
- [MDN — :where()](https://developer.mozilla.org/en-US/docs/Web/CSS/:where)
- Bramus, [The CSS :has() selector is way more than just a parent selector](https://www.bram.us/2021/12/21/the-css-has-selector/)

---

## 📝 Récap & checklist

- [ ] Je sais utiliser `:is()` pour factoriser
- [ ] Je sais utiliser `:where()` pour neutraliser la spécificité
- [ ] Je connais 3 cas d'usage `:has()` chez Dymasco
- [ ] Je sais que `:has()` / `:is()` prennent la spécificité du **plus spécifique** dans la liste

> 🎯 **Mantra du module** : *"Avant `:has()`, on ajoutait du JS. Maintenant, on écrit une ligne de CSS."*
