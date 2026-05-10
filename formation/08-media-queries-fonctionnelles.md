# Module 8 — Media Queries fonctionnelles

> ⏱️ **Durée** : 1h15 — **J2 après-midi, ouverture**
> 🧭 **Type** : module ergonomique (modes utilisateur, environnements physiques)
> 🎯 **Checkpoint** : `projet-fil-rouge/checkpoints/08-media-queries/overrides.css`

---

## 🎯 Objectif (en 1 phrase)

Adapter le dashboard aux **conditions réelles d'usage** : équipe nuit, opérateurs en gants tactiles, sensibilités visuelles, préférences animation, impression d'ordres de fabrication.

---

## ⚙️ Pour qui c'est utile chez Dymasco

Vos dashboards tournent dans des environnements **très variés** :

| Cas | Media query qui sauve |
|---|---|
| Équipe 3×8, équipe de nuit prend la lumière en pleine figure | `prefers-color-scheme: dark` |
| Opérateur avec gants → souris imprécise / tap pataud | `(hover: none) and (pointer: coarse)` |
| Stagiaire sensible aux animations, flicker disturbing | `prefers-reduced-motion: reduce` |
| Daltonien, contraste insuffisant | `prefers-contrast: more` |
| Imprimer un OF / un journal pour la main courante atelier | `@media print` |

Les media queries **fonctionnelles** ne posent pas la question "quelle taille d'écran ?" mais "**quelle préférence ou capacité de l'utilisateur ?**". C'est le module qui transforme un dashboard "qui marche" en dashboard **respectueux**.

---

## 📚 Concepts (30 min)

### 1. `prefers-color-scheme` — mode sombre (8 min)

```css
:root {
  color-scheme: light dark;       /* Module 7 */
  --ml-color-bg-app: #faf3e0;
  --ml-color-text:   #1f1f1f;
}

@media (prefers-color-scheme: dark) {
  :root {
    --ml-color-bg-app: oklch(20% 0.02 60);
    --ml-color-text:   oklch(95% 0.01 80);
  }
}
```

→ Le navigateur utilise les variables qui correspondent à la préférence OS.

#### Alternative `light-dark()` (cf. Module 7)

```css
:root {
  --ml-color-bg-app: light-dark(#faf3e0, oklch(20% 0.02 60));
}
```

Plus concis, mais Newly available — selon la cible client.

#### Cible Baseline

- `prefers-color-scheme` : **Widely available** depuis 2019. ✅.
- `light-dark()` : Newly available 2024. À arbitrer.

### 2. `prefers-reduced-motion` — désactiver les animations (5 min)

```css
.apriso-machine-card {
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}

@media (prefers-reduced-motion: reduce) {
  .apriso-machine-card {
    transition: none;
  }

  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

→ Pour les utilisateurs sensibles (vertiges, troubles vestibulaires, neurodivergence) ou simplement préférant un environnement calme.

> 💡 Le `!important` ici est exceptionnellement justifié : on **veut** écraser tout, peu importe la couche. C'est la valeur par défaut du **reset reduced-motion** documenté.

### 3. `prefers-contrast` — pour les daltoniens et basse vision (4 min)

```css
@media (prefers-contrast: more) {
  :root {
    --ml-color-text: oklch(10% 0 0);
    --ml-color-text-muted: oklch(30% 0 0);
    --ml-color-border: oklch(40% 0 0);
  }
  .apriso-machine-card { border-width: 2px; }
}
```

→ Augmente les contrastes pour les utilisateurs qui ont activé la préférence système.

Cible Baseline : Widely available 2023. ✅.

### 4. `hover` et `pointer` — tactile vs souris (8 min)

C'est **le** module pour Dymasco qui livre sur **tablettes opérateurs** (souvent avec **gants**).

```css
/* Souris fine (PC superviseur) — feedback hover OK */
@media (hover: hover) and (pointer: fine) {
  .apriso-machine-card:hover {
    box-shadow: var(--ml-shadow-2);
  }
}

/* Tactile coarse (tablette opérateur, gants) — feedback au tap */
@media (hover: none) and (pointer: coarse) {
  .apriso-machine-card:active {
    transform: scale(0.98);
  }

  /* Cibles tactiles plus grandes */
  .apriso-line-selector__select { min-block-size: 44px; }
  button, a { min-block-size: 44px; }
}
```

| Token | Sens |
|---|---|
| `hover: hover` | Le device a un vrai état de hover (souris) |
| `hover: none` | Pas de hover possible (tactile pur) |
| `pointer: fine` | Pointage précis (souris, stylet) |
| `pointer: coarse` | Pointage imprécis (doigt, gant) |

> ⚠️ Une cible tactile **doit** faire **au moins 44×44 px** (recommandation WCAG 2.5.5). Avec gants : 48-56 px conseillés.

#### Cible Baseline

`hover` / `pointer` : Widely available depuis ~ 2018. ✅.

### 5. `@media print` — l'impression d'OF (5 min)

Cas Dymasco : imprimer le journal d'événements de la nuit pour la main courante.

```css
@media print {
  /* Cacher les zones inutiles */
  .apriso-header__controls,
  .apriso-machine-grid,
  .apriso-footer__item--time {
    display: none;
  }

  /* Forcer le noir sur blanc, économie d'encre */
  :root {
    --ml-color-bg-app:   white;
    --ml-color-bg-cream: white;
    --ml-color-surface:  white;
    --ml-color-text:     black;
    --ml-color-border:   #999;
  }

  /* Étirer le journal en pleine page */
  .apriso-event-log {
    border: none;
    inline-size: 100%;
  }

  /* Casser proprement entre tableaux */
  .apriso-event-log__table tr { break-inside: avoid; }

  /* Afficher l'URL des liens */
  a::after {
    content: " (" attr(href) ")";
    color: #666;
  }

  /* Forcer fond couleurs si nécessaire */
  * { print-color-adjust: exact; }
}
```

#### `break-inside`, `break-before`, `break-after`

| Valeur | Sens |
|---|---|
| `avoid` | Ne pas couper l'élément entre deux pages |
| `page` | Forcer un saut de page avant/après |
| `auto` | Comportement par défaut |

---

## 🔍 Démonstration (15 min)

### Étape 1 — Mode sombre (5 min)

DevTools → "Emulate CSS prefers-color-scheme: dark" → bascule immédiate. Vérifier que toutes les surfaces, textes, bordures suivent.

### Étape 2 — Reduced motion (3 min)

DevTools → "Emulate CSS prefers-reduced-motion: reduce" → tester un hover sur card, vérifier que la transition est désactivée.

### Étape 3 — Tactile vs souris (4 min)

DevTools → "More tools → Sensors → Touch: Force enabled". Inspecter les media queries actives. Vérifier que `(hover: none)` matche.

### Étape 4 — Print preview (3 min)

DevTools → "Rendering → Emulate CSS media type: print", ou Cmd+P. Le dashboard se réduit au journal, en N&B, prêt à imprimer.

---

## 🛠️ Exercice fil rouge (25 min)

### Consigne

Reprendre le checkpoint Module 7 et :

1. **Mode sombre** : ajouter un bloc `@media (prefers-color-scheme: dark)` qui redéfinit les tokens surfaces / texte / borders en oklch sombres. Tester via DevTools.

2. **Reduced motion** : ajouter le bloc reset universal en `@media (prefers-reduced-motion: reduce)`. Vérifier qu'aucune transition ne se déclenche.

3. **Hover/pointer** :
   - Encapsuler le `:hover` des cards dans `@media (hover: hover) and (pointer: fine)`.
   - Ajouter dans `@media (hover: none) and (pointer: coarse)` :
     - `:active` avec `transform: scale(0.98)`.
     - `min-block-size: 44px` sur `select`, `button`, `a`.

4. **Contraste** : ajouter un bloc `@media (prefers-contrast: more)` qui assombrit le texte et épaissit les bordures.

5. **Print** : ajouter un bloc `@media print` qui :
   - Cache header controls, machine grid, footer time.
   - Force la palette en N&B.
   - Étire le journal.
   - `break-inside: avoid` sur les `<tr>`.

### Tests à faire

- DevTools : émulation `prefers-color-scheme: dark` → vérifier lisibilité.
- DevTools : émulation `prefers-reduced-motion: reduce` → vérifier que rien ne bouge au hover.
- DevTools : Cmd+P → preview impression du journal.

### Pièges fréquents

- ❌ Définir un mode sombre qui inverse **uniquement** background/color → bordures invisibles, ombres ratées. Toujours retoucher l'**ensemble** des tokens.
- ❌ Oublier le `!important` dans le reset reduced-motion → transitions résiduelles côté composants.
- ❌ Utiliser `@media (hover: hover)` sans le combiner avec `pointer: fine` → tablettes hybrides (Surface) matchent les deux. Mieux vaut être strict.
- ❌ `@media print` qui ne cache rien → journal au milieu du dashboard imprimé sur 4 pages.
- ❌ Oublier `print-color-adjust: exact` → certains backgrounds ne s'impriment pas.

### Corrigé attendu

Voir `projet-fil-rouge/checkpoints/08-media-queries/overrides.css`.

---

## 🚀 Pour aller plus loin

### Bonus 1 — `prefers-reduced-data`

```css
@media (prefers-reduced-data: reduce) {
  img { display: none; }
  video { autoplay: false; }
}
```

Cible Newly available, peu déployée. Mais utile en industrie / sites distants à connexion limitée.

### Bonus 2 — `forced-colors` (Windows High Contrast)

```css
@media (forced-colors: active) {
  .apriso-machine-card { border: 2px solid CanvasText; }
}
```

Pour les utilisateurs Windows en mode contraste forcé. Le navigateur impose ses propres couleurs ; on adapte.

### Bonus 3 — Combinaisons

```css
@media (prefers-color-scheme: dark) and (prefers-contrast: more) {
  :root { --ml-color-text: white; --ml-color-bg-app: black; }
}
```

Mode "extreme dark" pour utilisateurs qui cumulent les préférences.

### Ressources

- [MDN — prefers-color-scheme](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-color-scheme)
- [MDN — prefers-reduced-motion](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-reduced-motion)
- [MDN — pointer & hover](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/pointer)
- [MDN — @media print](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_paged_media)
- WCAG 2.5.5 — [Target Size](https://www.w3.org/WAI/WCAG21/Understanding/target-size.html)

---

## 📝 Récap & checklist

- [ ] Je connais `prefers-color-scheme` et je sais l'implémenter via tokens
- [ ] Je respecte `prefers-reduced-motion` partout où j'anime
- [ ] Je distingue souris fine / tactile coarse via `hover` + `pointer`
- [ ] Je connais le seuil **44×44 px** WCAG pour les cibles tactiles
- [ ] J'ai un bloc `@media print` pour mes vues imprimables (journaux, OF)
- [ ] J'utilise `break-inside: avoid` sur les lignes de tableau imprimées
- [ ] Je teste mes 4 modes via DevTools systématiquement avant livraison

> 🎯 **Mantra du module** : *"Le dashboard sait sur quel poste il tourne, et qui le regarde."*
