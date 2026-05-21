---
formateur:
  prerequis_formateur:
    - savoir que SEULES transform + opacity sont GPU-accelerated (pas de repaint)
    - savoir pourquoi transition: all = paresseux ET coûteux (recalcule layout)
    - currentColor + fill="currentColor" = 1 SVG asset, N teintes
    - aria-hidden="true" sur picto décoratif redondant (lecteur écran)
    - durées recommandées : hover 100-200ms / apparition 200-300ms
  ressources_demo_a_preparer:
    - checkpoint Module 8 ouvert
    - lucide.dev ouvert pour copier-coller icônes
    - DevTools Performance prêt (enregistrement frame rate)
    - 1 SVG triangle-alert prêt à coller dans HTML Laminoir Léon
  pitch_ouverture: >
    "Opérateur lit en 0,5 seconde. Badge texte 'Alerte' = trop lent. Picto rouge
    pulsé doucement = instantané. Et on tue transition: all qui sature les CPU
    tablette — uniquement transform + opacity, propre, performant."
  energie_attendue: posée — fin J2, finition visuelle = plaisir esthétique partagé
  duree_cible: 60 min — découpage 25 concepts / 15 démo / 15 exo / 5 récap
  variantes_timing:
    si_en_retard: zapper bonus @property + view-timeline (citer noms juste)
    si_en_avance: faire choisir pictos pour leur cas client dans lucide.dev
  points_a_marteler:
    - "JAMAIS transition: all — toujours cibler propriétés précises"
    - "transform + opacity = GPU-accelerated, pas de repaint"
    - "currentColor = magie SVG inline — 1 asset, N teintes contextuelles"
    - "animation pulse sobre — opacité picto SEUL, pas card entière"
    - "aria-hidden='true' sur picto décoratif (sinon lecteur écran double l'info)"
  pieges_apprenants:
    - animer box-shadow ou border au :hover → repaints, tablette saccade
    - oublier currentColor → fill="#xxx" hardcodé, perd teinte contexte
    - animer card ENTIÈRE au lieu du picto → distrayant, perf dégradée
    - durées > 300ms hover → impression de lourdeur
    - oublier que animation pulse héritera du prefers-reduced-motion Module 8
  questions_probables:
    - q: "SVG inline vs <img src.svg> ?"
      r: inline = stylable CSS (currentColor). External = cacheable, pas stylable.
    - q: "Sprite SVG sans pipeline build ?"
      r: possible mais lourd. Préférer inline ou Web Component (hors scope).
    - q: "@property pour animer variables, utilisable ?"
      r: Newly. OK Dymasco. Vérifier cible client.
    - q: "view-timeline scroll-driven ?"
      r: Newly. Joli mais pas critique. Citer, garder pour plus tard.
  transition_module_suivant: >
    "Visuel : posé. Module 10, dernier : on rend le dashboard LIVRABLE. Focus clavier
    visible, contrastes WCAG, perf optimisée. Et une check-list mise en prod à
    imprimer — celle que vous utiliserez avant chaque livraison."
---

# Module 9 — SVG & micro-animations d'état

> ⏱️ **Durée** : 1h00 — **J2 après-midi**
> 🧭 **Type** : module finition (pictos SVG inline, transitions ciblées, animation sobre)
> 🎯 **Checkpoint** : `projet-fil-rouge/checkpoints/09-svg-animations/overrides.css`

---

## 🎯 Objectif (en 1 phrase)

Remplacer les badges "texte seul" par des **pictos SVG d'état**, et passer des `transition: all` paresseuses à des **transitions ciblées sur `transform` et `opacity`** — performantes et respectueuses.

---

## ⚙️ Pour qui c'est utile chez Dymasco

Dans un dashboard d'atelier, **un opérateur lit en 0,5 seconde**. Un badge texte "Alerte" met plus de temps à être identifié qu'un picto rouge clignotant doucement.

Aujourd'hui dans `apriso-base.css` :

```css
.apriso-machine-card { transition: all 0.3s ease; }   /* ⚠️ "all" = toutes les props */
```

Et la card a juste un `<span>` "Alerte". Pas de feedback visuel rapide, et la transition repaint tout (`width`, `border`, `box-shadow`, `background`…) → saccades garanties sur tablette.

Après ce module :
- Badge enrichi d'un **picto SVG inline** colorisé via `currentColor`.
- Transition **uniquement** sur `transform` et `opacity` (les seules props GPU-accelerated).
- **Animation `pulse` sobre** sur les cards alert critique — discrète, désactivable via `prefers-reduced-motion` (Module 8).

---

## 📚 Concepts (25 min)

### 1. SVG inline + `currentColor` (8 min)

Un SVG inline dans le HTML peut être stylé via CSS, comme n'importe quel élément.

```html
<svg viewBox="0 0 24 24" class="apriso-status-icon" aria-hidden="true">
  <circle cx="12" cy="12" r="10" fill="currentColor" />
</svg>
```

```css
.apriso-status-icon { color: var(--ml-color-status-running); width: 1em; }
```

→ Le `fill="currentColor"` du SVG hérite de la `color` CSS. **Une icône, N teintes** selon le contexte.

#### Pourquoi `currentColor` est magique

```css
.apriso-machine-card--alert .apriso-status-icon { color: var(--ml-color-status-alert); }
.apriso-machine-card--running .apriso-status-icon { color: var(--ml-color-status-running); }
```

→ Aucun `fill` à dupliquer. Un seul SVG asset.

#### Cas d'usage Dymasco

- Pictos de statut machine (running, alert, maintenance, idle).
- Icônes de sévérité événement (critique, warning, info).
- Indicateur de tendance KPI (▲ ▼ —).

### 2. Patron picto SVG inline (5 min)

Pas de pipeline = pas de `<use href="">` distant. SVG **inline** dans le HTML, ou injecté via un Web Component (hors scope).

Bibliothèque utile : [Lucide](https://lucide.dev/), [Tabler](https://tabler.io/icons), [Heroicons](https://heroicons.com/) — copier-coller du SVG, et c'est parti.

```html
<svg class="apriso-status-icon" viewBox="0 0 24 24"
     fill="none" stroke="currentColor" stroke-width="2"
     stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
  <path d="M12 9v4" />
  <path d="M12 17h.01" />
  <circle cx="12" cy="12" r="10" />
</svg>
```

```css
.apriso-status-icon {
  width: 1em;
  height: 1em;
  vertical-align: -0.125em;
  flex: 0 0 auto;
}
```

### 3. Transitions ciblées (5 min)

#### Le vrai problème de `transition: all`

Quand le navigateur anime, il doit **recalculer le layout** pour les propriétés non-composées (`width`, `padding`, `border`…). Sur tablette, ça saccade.

Seules **deux** propriétés sont **GPU-accelerated** sans repaint :
- `transform` (translate, scale, rotate)
- `opacity`

```css
/* ❌ paresseux, peu performant */
.card { transition: all 0.3s ease; }

/* ✅ ciblé, performant */
.card {
  transition: transform 0.18s ease-out, opacity 0.18s ease-out, box-shadow 0.18s ease-out;
}
```

#### Conventions de durée

| Type d'interaction | Durée |
|---|---|
| Hover / focus | 100-200 ms |
| Apparition / disparition | 200-300 ms |
| Layout shift | À éviter |

#### `transition-property: discrete`

Récent : permet d'animer des propriétés discrètes (`display`, `visibility`). Newly available 2024. À garder en tête, mais hors scope ici.

### 4. Animations sobres : `pulse` sur alerte critique (4 min)

Pour signaler une alerte critique **sans agressivité** :

```css
@keyframes ml-pulse {
  0%, 100% { opacity: 1; }
  50%      { opacity: 0.55; }
}

.apriso-machine-card--alert .apriso-status-icon {
  animation: ml-pulse 1.6s ease-in-out infinite;
}
```

→ Pulsation lente, sur l'**opacité du picto seulement**. Désactivée automatiquement par le bloc `prefers-reduced-motion` du Module 8.

> ⚠️ **Évitez** `animation` sur la card entière (boîte, bordure, fond) — repaints constants.

### 5. `aria-hidden="true"` sur les pictos décoratifs (3 min)

Un picto **redondant** avec un libellé textuel doit être caché aux lecteurs d'écran :

```html
<span class="apriso-machine-card__status">
  <svg aria-hidden="true">...</svg>
  Alerte
</span>
```

Sinon le lecteur dit "image alerte" puis "alerte" → redondant. Module 10 reviendra sur l'A11y.

---

## 🔍 Démonstration (15 min)

### Étape 1 — Picto SVG injecté dans le badge (5 min)

Coller un SVG Lucide `triangle-alert` dans le `<span class="apriso-machine-card__status">` de Laminoir Léon. Styler :

```css
.apriso-machine-card__status .apriso-status-icon {
  width: 14px;
  margin-right: var(--ml-spacing-1);
}
```

→ Badge devient `[△] Alerte`, lisible en 0,3s.

### Étape 2 — Couleurs via `currentColor` (3 min)

```css
.apriso-machine-card--alert .apriso-machine-card__status {
  color: var(--ml-color-status-alert);
}
```

→ Picto + texte teintés ensemble.

### Étape 3 — Transition ciblée (3 min)

Remplacer `transition: all 0.3s ease` (côté framework, hérité) par `transition: transform 0.18s ease-out, box-shadow 0.18s ease-out` côté override. Hover → l'effet est plus net, plus rapide.

### Étape 4 — Animation pulse (4 min)

Ajouter le `@keyframes ml-pulse` + l'animation sur le picto des cards alert. Vérifier que l'animation **ne s'applique pas** sous `prefers-reduced-motion: reduce` (héritage Module 8).

---

## 🛠️ Exercice fil rouge (15 min)

### 📁 Code de départ

Partez du checkpoint Module 8 : [projet-fil-rouge/checkpoints/08-media-queries/overrides.css](../projet-fil-rouge/checkpoints/08-media-queries/overrides.css)

### 🎯 Résultat attendu

À la fin de l'implémentation, chaque badge de statut porte un **picto SVG inline** qui prend la couleur du contexte via `currentColor` (running → vert, alert → rouge…) et s'aligne sur la baseline du texte. Le picto des cards `--alert` **pulse** discrètement (opacité 1 ↔ 0.55, 1.6s) pour attirer l'œil sans saturer. Transitions cards : seulement `transform` + `box-shadow` (GPU, fluide). Sous `prefers-reduced-motion: reduce` (héritage M08) : pulse et transitions désactivés automatiquement. Lecteurs d'écran ignorent les SVG (`aria-hidden`).

### Consigne

Reprendre le checkpoint Module 8 et :

1. **Ajouter un picto SVG inline** dans chaque badge `.apriso-machine-card__status` du HTML (4 statuts : running, idle, alert, maintenance). Utiliser des icônes Lucide ou équivalent.

2. **Ajouter aria-hidden="true"** sur chaque SVG décoratif.

3. **Styler `.apriso-status-icon`** :
   - `width: 1em`, `height: 1em`.
   - `flex: 0 0 auto`.
   - `vertical-align: -0.125em` pour aligner sur la baseline du texte.

4. **Convertir transitions** :
   - Sur `.apriso-machine-card`, **remplacer** la transition héritée par `transition: transform 0.18s ease-out, box-shadow 0.18s ease-out`.

5. **Animation `ml-pulse`** :
   - Définir `@keyframes ml-pulse` (opacité 1 ↔ 0.55).
   - Appliquer sur `.apriso-machine-card--alert .apriso-status-icon` (1.6s, ease-in-out, infinite).

6. **Test reduced motion** : DevTools → émulation `prefers-reduced-motion: reduce`. Vérifier que l'animation est suspendue (héritage du Module 8).

### Pièges fréquents

- ❌ Animer `box-shadow` ou `border` sur `:hover` → repaints. Préférer `transform: translateY(-2px)` ou `scale()`.
- ❌ Oublier `currentColor` → couleurs SVG hardcodées en `<svg fill="#xxx">`, perdent la teinte du contexte.
- ❌ Animer la **card entière** au lieu du picto → distrayant, performance dégradée.
- ❌ Oublier `aria-hidden` → lecteurs d'écran annoncent un élément "image vide".
- ❌ Durées > 300 ms sur du hover → impression de lourdeur.

### Corrigé attendu

Voir [projet-fil-rouge/checkpoints/09-svg-animations/overrides.css](../projet-fil-rouge/checkpoints/09-svg-animations/overrides.css).

---

## 🚀 Pour aller plus loin

### Bonus 1 — `@property` pour animer des variables

```css
@property --ring-angle {
  syntax: "<angle>";
  initial-value: 0deg;
  inherits: false;
}

.spinner { animation: rotate 1s linear infinite; }
@keyframes rotate { to { --ring-angle: 360deg; } }
```

→ Permet d'animer des **variables CSS typées** comme s'il s'agissait de propriétés natives. Newly available.

### Bonus 2 — `view-timeline` (scroll-driven animations)

```css
.apriso-machine-card {
  view-timeline-name: --card;
  animation-timeline: --card;
  animation-name: fade-in;
}
```

→ Animations pilotées par le scroll. Newly available — à surveiller.

### Bonus 3 — SVG sprite si beaucoup d'icônes

Pas notre cas (pas de build), mais à connaître pour des projets plus gros : un seul `<svg>` contenant N `<symbol>`, référencés via `<use href="#id">`.

### Ressources

- [Lucide — pictos SVG open source](https://lucide.dev/)
- [MDN — currentColor](https://developer.mozilla.org/en-US/docs/Web/CSS/color_value#currentcolor_keyword)
- [MDN — transition](https://developer.mozilla.org/en-US/docs/Web/CSS/transition)
- [MDN — @keyframes](https://developer.mozilla.org/en-US/docs/Web/CSS/@keyframes)
- Paul Lewis, [High Performance Animations](https://web.dev/articles/animations-guide)

---

## 📝 Récap & checklist

- [ ] Mes pictos SVG sont **inline** et utilisent `currentColor`
- [ ] Je n'écris **plus** `transition: all`
- [ ] Mes transitions ciblent `transform` / `opacity` / `box-shadow`
- [ ] Mes durées d'interaction sont **100-300 ms** max
- [ ] J'utilise `aria-hidden="true"` sur les pictos décoratifs
- [ ] Mes animations critiques (`pulse`) sont **sobres** et désactivées sous `prefers-reduced-motion`

> 🎯 **Mantra du module** : *"Une animation ne sert pas à décorer. Elle sert à informer."*
