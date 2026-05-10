# Module 7 — Couleurs modernes & thèmes

> ⏱️ **Durée** : 1h00 — **J2 matin, clôture**
> 🧭 **Type** : module identité visuelle
> 🎯 **Checkpoint** : `projet-fil-rouge/checkpoints/07-couleurs/overrides.css`

---

## 🎯 Objectif (en 1 phrase)

Passer la palette de **hex statique** à **`oklch` perceptuel + `color-mix()` dynamique**, et préparer le terrain `color-scheme` pour le mode sombre (Module 8).

---

## ⚙️ Pour qui c'est utile chez Dymasco

L'identité Mamie Lulu n'est pas finalisée. Demain Lulu va dire "le orange est un peu agressif sur les écrans de nuit, on peut foncer un peu ?". Aujourd'hui : vous repassez sur **N occurrences hex**, vous créez 3 variantes hover, 3 variantes alert, etc.

Avec `oklch` et `color-mix()` :
- **Une variable `--ml-color-brand-primary`** pilote tout.
- Les variantes (hover, focus, alert background) sont **dérivées dynamiquement**.
- Modifier la teinte = changer un seul nombre, tout suit **avec une luminance perceptive cohérente**.

Aujourd'hui vous avez aussi :
- 3 verts différents pour "running" (`#4caf50`, `#2e7d32`, `#1b5e20`) parce qu'on a tâtonné.
- Pas de système, pas de cohérence, refactor pénible.

---

## 📚 Concepts (25 min)

### 1. Pourquoi `oklch` plutôt que `#hex` / `rgb` / `hsl` ? (8 min)

Trois problèmes des espaces colorimétriques classiques :

#### a) Luminance perçue ≠ luminance calculée

En `hsl`, deux couleurs avec **la même `lightness`** peuvent avoir des luminances perçues très différentes. Ex : `hsl(60, 100%, 50%)` (jaune vif) paraît **bien plus clair** que `hsl(240, 100%, 50%)` (bleu sombre), alors que la valeur `lightness` est identique.

Conséquence : impossible de bâtir une échelle "claire → foncée" cohérente sans tâtonner.

#### b) Mauvaise interpolation

Mélanger deux couleurs hex (ex : pour générer un hover) donne souvent **du gris** ou des teintes ternes au passage. Le mix passe par l'espace RGB qui n'est pas perceptuel.

#### c) Gamut limité

`hex` / `rgb` ne savent décrire que les couleurs sRGB. Les écrans modernes (P3) en affichent davantage. `oklch` accède à tout le gamut.

#### `oklch` en pratique

```css
oklch(L C H / alpha)
```

| Paramètre | Sens | Plage |
|---|---|---|
| `L` (lightness) | Luminance **perceptive** | 0% (noir) → 100% (blanc) |
| `C` (chroma) | Saturation, sans plafond fixe | 0 (gris) → ~0.4 (très saturé) |
| `H` (hue) | Teinte | 0 → 360 (degrés) |

Exemple :

```css
:root {
  --ml-color-brand-primary: oklch(58% 0.18 35);    /* tomate Mamie Lulu */
  --ml-color-brand-accent:  oklch(78% 0.16 80);    /* jaune œuf */
}
```

→ Si je veux une **variante plus claire** : je change juste `L` (`72%` au lieu de `58%`). Si je veux une **teinte voisine** : je change juste `H`.

#### Cible Baseline

`oklch` : **Widely available** depuis 2023 (Chrome/Edge 111+, Safari 15.4+, Firefox 113+). Pour Dymasco : ✅.

### 2. `color-mix()` — la dérivation dynamique (10 min)

```css
color-mix(in <color-space>, <color-1> <pct?>, <color-2> <pct?>)
```

Génère une couleur intermédiaire entre deux couleurs.

```css
:root {
  --ml-color-brand-primary: oklch(58% 0.18 35);
}

/* Variantes dérivées */
.btn:hover {
  background: color-mix(in oklab, var(--ml-color-brand-primary), black 12%);
  /* 88% de la couleur + 12% de noir → version assombrie */
}

.btn:active {
  background: color-mix(in oklab, var(--ml-color-brand-primary), black 22%);
}

.alert {
  background: color-mix(in oklab, var(--ml-color-status-alert), white 88%);
  /* fond très clair de la teinte alert */
}
```

#### Espaces de mix utiles

| Espace | Usage |
|---|---|
| `in oklab` | Le plus perceptuellement correct. **Défaut recommandé.** |
| `in oklch` | Idem, mais en coordonnées polaires (utile pour interpoler la teinte) |
| `in srgb` | Mix RGB classique — moins joli, mais plus rapide |

> 🎯 **Pattern Mamie Lulu** : tous les hover/active/alert/disabled sont dérivés via `color-mix()` à partir des **5 couleurs de base**. Aucune variante hex en dur.

#### Cible Baseline

`color-mix()` : **Widely available** depuis 2023. ✅.

### 3. `color-scheme` — préparation mode sombre (4 min)

```css
:root {
  color-scheme: light dark;       /* l'app sait gérer les deux */
}
```

→ Le navigateur :
- Adapte les **scrollbars natives** (claires en light, sombres en dark).
- Adapte les **inputs natifs** par défaut.
- Permet à `prefers-color-scheme` de prendre effet sur les composants UA.

Ce module **annonce** le sujet, **Module 8** l'implémente avec `@media (prefers-color-scheme: dark)`.

### 4. Les `light-dark()` — le pattern moderne (3 min)

```css
:root {
  color-scheme: light dark;
  --ml-color-bg-app: light-dark(#faf3e0, #1a1815);
}
```

→ Une seule déclaration, le navigateur choisit selon le mode courant.

#### Cible Baseline

`light-dark()` : **Newly available** (2024). À utiliser avec `@supports` si cible mixte.

---

## 🔍 Démonstration (15 min)

**Live** : refondre les tokens couleurs Mamie Lulu en oklch + color-mix.

### Étape 1 — Convertir hex → oklch (5 min)

Outil : [oklch.com](https://oklch.com) (picker visuel). Coller `#c2410c` → obtient `oklch(54% 0.17 38)`. Expliquer chaque paramètre.

### Étape 2 — Dériver les variantes (5 min)

```css
@layer overrides {
  :root {
    --ml-color-brand-primary: oklch(54% 0.17 38);
    --ml-color-brand-primary-hover:    color-mix(in oklab, var(--ml-color-brand-primary), black 10%);
    --ml-color-brand-primary-active:   color-mix(in oklab, var(--ml-color-brand-primary), black 20%);
    --ml-color-brand-primary-disabled: color-mix(in oklab, var(--ml-color-brand-primary), white 50%);
  }
}
```

→ DevTools : modifier la valeur de `--ml-color-brand-primary` à la volée → tout suit.

### Étape 3 — `color-scheme` sur `:root` (2 min)

Ajouter `color-scheme: light dark`. Observer dans DevTools → scrollbar de la sidebar passe en sombre quand on bascule la préférence OS.

### Étape 4 — Surfaces dérivées (3 min)

Showcase : générer toute la palette de surfaces depuis 1 brand color :

```css
--ml-color-surface-1: color-mix(in oklab, var(--ml-color-brand-primary), white 96%);
--ml-color-surface-2: color-mix(in oklab, var(--ml-color-brand-primary), white 90%);
--ml-color-surface-3: color-mix(in oklab, var(--ml-color-brand-primary), white 80%);
```

→ Trois fonds chauds Mamie Lulu, déduits, parfaitement harmonisés.

---

## 🛠️ Exercice fil rouge (15 min)

### Consigne

Reprendre le checkpoint Module 6 et :

1. **Convertir tous les `--ml-color-*` en `oklch()`**. Outil : oklch.com pour la conversion. Approximations OK pour l'exercice.

2. **Créer les variantes dérivées** pour `--ml-color-brand-primary` :
   - `--ml-color-brand-primary-hover` (assombrie 10%)
   - `--ml-color-brand-primary-active` (assombrie 20%)

3. **Backgrounds dérivés** pour les statuts alert et critical :
   - `--ml-color-status-alert-bg` = mix avec white 90%.
   - `--ml-color-status-critical-bg` = mix avec white 92%.
   - **Remplacer** les `color-mix()` inline qui traînent dans le checkpoint Module 6 par ces tokens.

4. **`color-scheme`** sur `:root` : `color-scheme: light dark`.

5. **Bonus** : token `--ml-color-bg-app` en `light-dark(#faf3e0, oklch(20% 0.02 60))` (préparation Module 8).

### Pièges fréquents

- ❌ Convertir mal de hex à oklch → différence visible. Toujours vérifier visuellement.
- ❌ Mettre `color-mix()` inline partout (oubli de tokenisation) → on perd l'avantage centralisé.
- ❌ Oublier l'espace de mix (`color-mix(<color-1>, <color-2>)` sans `in oklab`) → erreur silencieuse, valeur ignorée.
- ❌ `oklch()` avec L > 100% ou C négatif → valeur invalide, propriété ignorée silencieusement.

### Corrigé attendu

Voir `projet-fil-rouge/checkpoints/07-couleurs/overrides.css`.

---

## 🚀 Pour aller plus loin

### Bonus 1 — Échelle complète depuis une teinte

```css
:root {
  --hue: 35;
  --ml-color-brand-50:  oklch(96% 0.02 var(--hue));
  --ml-color-brand-100: oklch(92% 0.04 var(--hue));
  --ml-color-brand-500: oklch(58% 0.18 var(--hue));
  --ml-color-brand-700: oklch(42% 0.16 var(--hue));
  --ml-color-brand-900: oklch(24% 0.10 var(--hue));
}
```

→ Toute la gamme depuis **un seul angle de teinte**. Changer `--hue: 220` → tout devient bleu.

### Bonus 2 — `color-contrast()`

```css
color: color-contrast(var(--bg) vs white, black);
```

→ Sélectionne automatiquement la couleur la plus contrastée parmi une liste. **Limited availability** aujourd'hui, à surveiller.

### Bonus 3 — Tokens "intentionnels" vs "primitifs"

Pattern à 2 niveaux :
- **Primitifs** : `--ml-orange-500`, `--ml-cream-100` (palette brute)
- **Intentionnels** : `--ml-color-brand-primary` → `var(--ml-orange-500)`

Permet de changer la palette sans toucher au code applicatif.

### Ressources

- [oklch.com](https://oklch.com) — picker visuel
- [MDN — oklch()](https://developer.mozilla.org/en-US/docs/Web/CSS/color_value/oklch)
- [MDN — color-mix()](https://developer.mozilla.org/en-US/docs/Web/CSS/color_value/color-mix)
- [MDN — color-scheme](https://developer.mozilla.org/en-US/docs/Web/CSS/color-scheme)
- Andrey Sitnik, [Better than HSL: Oklch in CSS](https://evilmartians.com/chronicles/oklch-in-css-why-quit-rgb-hsl)

---

## 📝 Récap & checklist

- [ ] Je sais lire `oklch(L C H)` et expliquer chaque paramètre
- [ ] Je sais que `oklch` est **perceptuellement uniforme** (changements visibles cohérents)
- [ ] Je dérive mes variantes hover/active/disabled via `color-mix()`
- [ ] Je précise toujours l'espace de mix : `in oklab` (recommandé)
- [ ] Je déclare `color-scheme: light dark` sur `:root`
- [ ] Mes tokens couleurs sont **5-10 valeurs sources**, tout le reste est dérivé

> 🎯 **Mantra du module** : *"Une teinte source. Un nombre à changer. Tout suit."*
