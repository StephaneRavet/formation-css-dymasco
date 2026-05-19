---
formateur:
  prerequis_formateur:
    - savoir naviguer DevTools "Rendering" pour émuler prefers-* en live
    - connaître seuil WCAG 2.5.5 = 44×44px (gants = 48-56px conseillés)
    - savoir pourquoi !important justifié dans reset reduced-motion (override TOUT)
    - distinguer hover:hover/none ET pointer:fine/coarse (4 combinaisons possibles)
    - forced-colors + couleurs système (Canvas, CanvasText, Highlight) + forced-color-adjust
  ressources_demo_a_preparer:
    - checkpoint Module 7 ouvert
    - DevTools → Rendering ouvert (émulation prefers-color-scheme, reduced-motion, forced-colors)
    - DevTools → Sensors prêt (Touch: Force enabled)
    - poste Windows avec profil HC prêt (idéal — fallback DevTools sinon)
  pitch_ouverture: >
    "Équipe nuit qui prend la lumière en pleine figure. Opérateur avec gants tactiles
    qui galère sur un select de 32px. apprenant sensible aux animations. Site qui
    impose Windows High Contrast pour l'accessibilité — badge alerte rouge devient
    rectangle noir indiscernable. Ce module = passer de 'dashboard qui marche' à
    'dashboard respectueux'."
  energie_attendue: bonne — début aprem J2, post-déjeuner, sujets concrets/empathiques engagent
  duree_cible: 75 min — découpage 30 concepts / 15 démo / 25 exo / 5 récap
  variantes_timing:
    si_en_retard: zapper bonus prefers-reduced-data + print + combinaisons
    si_en_avance: faire imaginer 1 cas usage hover/pointer dans LEUR contexte client
  points_a_marteler:
    - "prefers-* = question à l'UTILISATEUR, pas à l'écran"
    - "44×44px MINIMUM tactile WCAG — gants atelier = 48-56px"
    - "!important justifié dans reset reduced-motion (override toutes couches)"
    - "hover: hover ET pointer: fine — combiner strict pour tablettes hybrides Surface"
    - "forced-colors: HC Windows écrase couleurs → restituer bordures via CanvasText + forced-color-adjust ciblé sur badges"
  pieges_apprenants:
    - dark mode = inverser juste bg/color → bordures et ombres invisibles
    - hover:hover sans pointer:fine → tablettes hybrides matchent les deux
    - forced-color-adjust: none en wildcard → HC ne sert plus à rien
    - tester forced-colors uniquement DevTools → certains comportements système (scrollbars, focus) non émulés
    - sticky+dark mode mal pensé → header sticky trop transparent en dark
  questions_probables:
    - q: "light-dark() vs @media prefers-color-scheme ?"
      r: light-dark() plus concis. @media plus de contrôle (tokens N>2 valeurs).
    - q: "forced-colors c'est quoi vs prefers-contrast ?"
      r: Windows High Contrast = navigateur impose ses couleurs. On adapte borders.
    - q: "Peut-on tester sans vraie tablette ?"
      r: oui — DevTools Sensors → Touch Force enabled. Imparfait mais utile.
    - q: "prefers-reduced-data, déjà supporté ?"
      r: Newly, peu déployé. Citer, ne pas planifier dessus.
  transition_module_suivant: >
    "L'expérience utilisateur respecte les préférences. Module 9 : finition visuelle.
    Pictos SVG inline + currentColor, transitions ciblées GPU, animation pulse sobre
    sur alertes. 1h pour passer 'qui marche' à 'qui informe en 0,3 seconde'."
---

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
| apprenant sensible aux animations, flicker disturbing | `prefers-reduced-motion: reduce` |
| Daltonien, contraste insuffisant | `prefers-contrast: more` |
| Site qui impose Windows High Contrast (DSI accessibilité) — badges status écrasés | `(forced-colors: active)` |

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
  .apriso-line-selector__select { min-height: 44px; }
  button, a { min-height: 44px; }
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

### 5. `forced-colors` — Windows High Contrast Mode (5 min)

**Cas Dymasco réel — pas une niche** : sur certains sites industriels, la DSI **impose** le profil Windows High Contrast (HC) sur les postes d'atelier pour des raisons d'accessibilité (opérateurs/superviseurs avec troubles visuels, environnement avec éblouissement lumineux fort, conformité réglementaire interne). Quand HC est actif, Windows **écrase les couleurs CSS** par sa palette système :

- Les `background-color` sont **remplacés** par `Canvas` (souvent noir).
- Les `border` peuvent **disparaître** si la couleur n'est pas critique pour le navigateur.
- Le résultat sur Apriso unlayered + dashboard coloré : un badge "alerte" rouge devient un rectangle noir indiscernable d'un badge "running" vert. **Régression critique en prod.**

C'est pour ça que `forced-colors` mérite une section dédiée chez Dymasco — pas un bonus.

```css
@media (forced-colors: active) {
  /* Restituer la sémantique via la BORDURE et le PATTERN, pas la couleur */
  .apriso-machine-card {
    border: 2px solid CanvasText;
    background: Canvas;
  }

  /* Status sémantique → préserver couleur uniquement sur l'indicateur critique */
  .apriso-machine-card--alert .apriso-machine-card__status {
    forced-color-adjust: none;   /* on demande à Windows de NOUS LAISSER */
    background: var(--ml-color-status-alert);
    color: white;
  }

  /* Focus ring : utiliser Highlight (la couleur de sélection système) */
  :focus-visible {
    outline: 3px solid Highlight;
    outline-offset: 2px;
  }
}
```

#### Couleurs système CSS (à connaître)

Quand HC est actif, on **n'écrit plus** `color: #1f1f1f` — on écrit `color: CanvasText`. Le navigateur fait la traduction selon le thème HC choisi par l'utilisateur (High Contrast Black, High Contrast White, High Contrast 1/2…).

| Mot-clé | Sens |
| --- | --- |
| `Canvas` | Fond de l'application |
| `CanvasText` | Texte par défaut |
| `LinkText` / `VisitedText` | Liens |
| `ButtonFace` / `ButtonText` | Boutons |
| `Highlight` / `HighlightText` | Sélection / focus |
| `GrayText` | Texte désactivé |

#### `forced-color-adjust` — l'échappatoire ciblée

`forced-color-adjust: none` dit au navigateur "**ne touche pas** à mes couleurs ici". À utiliser **uniquement** sur les éléments où la couleur **est l'information** (badge status critique, indicateur visuel safety). Jamais en wildcard `*` — sinon HC ne sert plus à rien.

#### Tester localement

- **Windows** : `Paramètres → Accessibilité → Thèmes de contraste → Aquatique/Désert/...`. Bascule immédiate.
- **DevTools (Chrome/Edge)** : `Rendering → Emulate CSS media feature forced-colors: active`. Suffit pour la démo formation, pas pour la validation finale.

#### Cible Baseline (forced-colors)

`forced-colors` : **Widely available** depuis 2023. ✅. `forced-color-adjust` : idem.

---

## 🔍 Démonstration (15 min)

### Étape 1 — Mode sombre (5 min)

DevTools → "Emulate CSS prefers-color-scheme: dark" → bascule immédiate. Vérifier que toutes les surfaces, textes, bordures suivent.

### Étape 2 — Reduced motion (3 min)

DevTools → "Emulate CSS prefers-reduced-motion: reduce" → tester un hover sur card, vérifier que la transition est désactivée.

### Étape 3 — Tactile vs souris (4 min)

DevTools → "More tools → Sensors → Touch: Force enabled". Inspecter les media queries actives. Vérifier que `(hover: none)` matche.

### Étape 4 — Forced colors / Windows HC (3 min)

DevTools → "Rendering → Emulate CSS media feature `forced-colors: active`". Avant : badges status (running/alerte) deviennent indiscernables (rectangles noirs). Après application du bloc `@media (forced-colors: active)` : bordures restituées via `CanvasText`, badge alerte conserve son rouge via `forced-color-adjust: none`. Bascule en live + comparaison avant/après = **effet pédagogique fort**.

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
     - `min-height: 44px` sur `select`, `button`, `a`.

4. **Contraste** : ajouter un bloc `@media (prefers-contrast: more)` qui assombrit le texte et épaissit les bordures.

5. **Forced colors (Windows HC)** : ajouter un bloc `@media (forced-colors: active)` qui :
   - Restaure les bordures via `CanvasText` sur cards / containers (Apriso les laissait à `background` only).
   - Préserve la sémantique couleur des badges status critiques via `forced-color-adjust: none` ciblé.
   - Met le focus ring sur `Highlight` (couleur système).
   - Tester via DevTools `Rendering → forced-colors: active`, vérifier que badge alerte reste rouge et que cards "running" vs "alert" restent **distinguables**.

### Tests à faire

- DevTools : émulation `prefers-color-scheme: dark` → vérifier lisibilité.
- DevTools : émulation `prefers-reduced-motion: reduce` → vérifier que rien ne bouge au hover.
- DevTools : Cmd+P → preview impression du journal.

### Pièges fréquents

- ❌ Définir un mode sombre qui inverse **uniquement** background/color → bordures invisibles, ombres ratées. Toujours retoucher l'**ensemble** des tokens.
- ❌ Oublier le `!important` dans le reset reduced-motion → transitions résiduelles côté composants.
- ❌ Utiliser `@media (hover: hover)` sans le combiner avec `pointer: fine` → tablettes hybrides (Surface) matchent les deux. Mieux vaut être strict.
- ❌ `forced-color-adjust: none` en wildcard `*` → HC ne sert plus à rien. Cibler uniquement les éléments où la **couleur est l'information** (badge alerte critique).
- ❌ Tester `forced-colors` uniquement en DevTools sans valider sur un vrai poste Windows HC → certains comportements (scrollbars, focus système) ne s'émulent pas.
- ❌ Oublier `:focus-visible { outline: 3px solid Highlight }` en HC → focus invisible, navigation clavier perdue.

### Corrigé attendu

Voir `projet-fil-rouge/checkpoints/08-media-queries/overrides.css`.

---

## 🚀 Pour aller plus loin

### Bonus 1 — `prefers-reduced-data`

```css
@media (prefers-reduced-data: reduce) {
  img { display: none; }
  video { display: none; }   /* `autoplay` est un attribut HTML, pas une propriété CSS — on cache l'élément */
}
```

Cible Newly available, peu déployée. Mais utile en industrie / sites distants à connexion limitée.

### Bonus 2 — `@media print` (cas marginal Dymasco)

Historiquement traité comme une feature centrale. **En pratique chez Dymasco** : dashboards consultés en continu sur écran atelier, jamais imprimés. La main courante papier existe rarement et passe par d'autres outils (export PDF métier, edition Apriso).

À garder en tête uniquement si le client demande explicitement une vue imprimable :

```css
@media print {
  .apriso-header__controls,
  .apriso-machine-grid { display: none; }

  :root {
    --ml-color-bg-app: white;
    --ml-color-text:   black;
  }

  .apriso-event-log__table tr { break-inside: avoid; }
}
```

`print-color-adjust: exact` à cibler **uniquement** sur les badges sémantiques (jamais en wildcard `*` — surconsommation d'encre). Émulation DevTools : `Rendering → Emulate CSS media type: print`.

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
- [ ] J'ai un bloc `@media (forced-colors: active)` qui restitue les bordures via `CanvasText` et préserve les badges status critiques via `forced-color-adjust: none`
- [ ] Je sais que `@media print` est un cas marginal Dymasco (cf. bonus) — pas un livrable par défaut
- [ ] J'utilise `break-inside: avoid` sur les lignes de tableau imprimées
- [ ] Je teste mes 4 modes via DevTools systématiquement avant livraison

> 🎯 **Mantra du module** : *"Le dashboard sait sur quel poste il tourne, et qui le regarde."*
