---
formateur:
  prerequis_formateur:
    - savoir color-mix(in oklab, ...) — espace par défaut recommandé
    - savoir que color-mix marche avec SOURCE hex (pas besoin de stocker en oklch)
    - distinguer color-scheme (UA hints) vs light-dark() (valeur conditionnelle)
    - savoir basculer prefers-color-scheme dans DevTools (Rendering)
    - oklch + color-mix widely 2023 / light-dark widely 2024 (vérifier au jour J)
  ressources_a_preparer:
    - checkpoint Module 6 ouvert
    - rappeler aux apprenants : DevTools pour modifier --ml-color-brand-primary à la volée
    - rappeler : DevTools Rendering pour bascule prefers-color-scheme light/dark
  pitch_ouverture: >
    "Lulu vous appelle : 'le orange est trop agressif sur les écrans de nuit,
    foncez-le un peu'. Avant : 47 occurrences hex à modifier. Après ce module :
    UNE seule variable, tout suit. Et bonus : un bouton light/dark qui marche
    en 3 lignes."
  energie_attendue: posée — clôture J2 matin, juste avant déjeuner
  duree_cible: 45 min — découpage 20 concepts / 22 exo (pas de démo séparée, ils testent direct) / 3 récap
  variantes_timing:
    si_en_retard: zapper bonus échelle hue + tokens primitifs/intentionnels
    si_en_avance: faire dériver palette hover/active sur LEUR charte client réelle
  points_a_marteler:
    - "color-mix(in oklab, HEX, ...) — pas besoin d'oklch côté source, ça marche avec ta charte client"
    - "TOUJOURS préciser l'espace : color-mix(in oklab, ...) — sans 'in X' c'est ignoré silencieusement"
    - "5-10 couleurs SOURCES (hex client), tout le reste DÉRIVÉ via color-mix"
    - "light-dark() + color-scheme = theme switcher complet en 2 lignes"
    - "oklch côté SOURCE = construction palette from scratch (rare). Côté MIX = oui (in oklab)"
  pieges_apprenants:
    - color-mix sans "in oklab" → erreur silencieuse, valeur ignorée
    - color-mix inline partout au lieu de tokeniser → perd avantage centralisé
    - light-dark() sans color-scheme déclaré → fonctionne mais pas optimal côté UA
    - confondre prefers-color-scheme (préférence user) et color-scheme (déclaration app)
  questions_probables:
    - q: "Pourquoi pas tout en oklch côté source ?"
      r: ça marche, mais charte client = hex 99% du temps. color-mix(in oklab, hex…) suffit. oklch utile QUAND tu construis la palette toi-même.
    - q: "color-mix vs sass mix(), différence ?"
      r: runtime + perceptual (oklab). Sass = build-time, espace RGB classique. Pas de pipeline build chez Dymasco → color-mix natif gagne.
    - q: "P3 / gamut large, on en bénéficie quand ?"
      r: bonus écrans modernes. Hors sujet écran atelier industriel sRGB.
    - q: "color-contrast() utilisable ?"
      r: Limited availability. Trop tôt pour Dymasco.
    - q: "Combien de tokens primitifs ?"
      r: 5-10 max. Au-delà = redondance, signal qu'il faut dériver via color-mix.
  transition_module_suivant: >
    "Couleurs + theme switcher : système posé. Module 8 : on adapte aux conditions
    RÉELLES — équipe nuit, opérateurs gants tactiles, sensibilités visuelles,
    Windows High Contrast. Les media queries fonctionnelles transforment 'qui
    marche' en 'respectueux'."
---

# Module 7 — Couleurs modernes & thèmes

> ⏱️ **Durée** : 0h45 — **J2 matin, clôture**
> 🧭 **Type** : module identité visuelle + theme switcher
> 🎯 **Checkpoint** : `projet-fil-rouge/checkpoints/07-couleurs/overrides.css`

---

## 🎯 Objectif (en 1 phrase)

Passer la palette de **hex éparpillé** à **5-10 sources hex + dérivations dynamiques `color-mix()`**, et préparer un **theme switcher light/dark** via `color-scheme` + `light-dark()`.

---

## ⚙️ Pour qui c'est utile chez Dymasco

Réalité quotidienne intégrateur Apriso :

- Le client envoie sa charte en **hex** (`#c2410c`, `#2e7d32`…). Jamais en oklch. Vous le saviez.
- Vous devez en tirer : variantes hover, active, disabled, backgrounds atténués, états alert…
- Aujourd'hui : 3 verts différents pour "running" (`#4caf50`, `#2e7d32`, `#1b5e20`) parce qu'on a tâtonné. Pas de système.
- Demain Lulu dit "fonce l'orange de 10%" → 47 occurrences à toucher.

**`color-mix()` règle 95 % du problème** — et accepte directement votre hex client en entrée. Pas besoin de tout convertir en oklch (l'espace de **mélange** est `in oklab`, mais les **sources** restent vos hex).

`light-dark()` règle l'autre 5 % : theme switcher complet en 2 lignes.

`oklch()` est une option pour construire une palette **from scratch** — citée, pas centrale.

---

## 📚 Concepts (20 min)

### 1. `color-mix()` — la dérivation dynamique (10 min)

```css
color-mix(in <color-space>, <color-1> <pct?>, <color-2> <pct?>)
```

Génère une couleur intermédiaire entre deux couleurs, **avec n'importe quel format de source** (hex, rgb, hsl, oklch, named colors…).

```css
:root {
  --ml-color-brand-primary: #c2410c;   /* charte client en hex, tel quel */
}

/* Variantes dérivées — sources hex, mix en oklab */
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

#### Pourquoi `in oklab` et pas `in srgb` ?

Le **mix** en RGB classique passe souvent par des teintes ternes / grisâtres au milieu (artefact mathématique de l'espace RGB non-perceptuel). En `oklab`, l'interpolation est perceptuellement uniforme : le milieu **ressemble** à un milieu. Différence visible surtout sur les mélanges entre couleurs saturées.

#### Espaces de mix utiles

| Espace | Usage |
| --- | --- |
| `in oklab` | Le plus perceptuellement correct. **Défaut recommandé.** |
| `in oklch` | Idem, coordonnées polaires (utile pour interpoler la teinte sur un cercle chromatique) |
| `in srgb` | Mix RGB classique — moins joli, à éviter sauf raison spécifique |

> ⚠️ **Piège** : oublier `in <space>` → `color-mix(var(--a), var(--b))` est **invalide** et la propriété est ignorée silencieusement. Toujours préciser.

#### Pattern Mamie Lulu

```css
@layer overrides {
  :root {
    /* SOURCES — charte client (hex tel quel) */
    --ml-color-brand-primary:      #c2410c;
    --ml-color-brand-accent:       #f59e0b;
    --ml-color-status-running:     #2e7d32;
    --ml-color-status-alert:       #f57c00;
    --ml-color-status-maintenance: #1976d2;

    /* DÉRIVÉS — color-mix tokenisé (pas inline dispersé) */
    --ml-color-brand-primary-hover:    color-mix(in oklab, var(--ml-color-brand-primary), black 10%);
    --ml-color-brand-primary-active:   color-mix(in oklab, var(--ml-color-brand-primary), black 20%);
    --ml-color-brand-primary-disabled: color-mix(in oklab, var(--ml-color-brand-primary), white 50%);

    --ml-color-status-alert-bg:        color-mix(in oklab, var(--ml-color-status-alert), white 90%);
    --ml-color-status-running-bg:      color-mix(in oklab, var(--ml-color-status-running), white 92%);
  }
}
```

→ 5 couleurs sources, ~10 dérivées. **Tout le reste de l'app utilise ces tokens.** Modifier `--ml-color-brand-primary` = tout suit (variantes incluses).

#### Cible Baseline

`color-mix()` : **Widely available** depuis 2023. ✅.

### 2. `color-scheme` + `light-dark()` — theme switcher (8 min)

#### `color-scheme` — annonce au navigateur

```css
:root {
  color-scheme: light dark;       /* l'app sait gérer les deux */
}
```

Effets immédiats :

- Les **scrollbars natives** suivent (claires en light, sombres en dark).
- Les **inputs natifs** (select, date picker…) adaptent leur thème UA.
- `prefers-color-scheme` peut prendre effet sur les composants navigateur sans CSS supplémentaire.

Coût : **1 ligne**. Toujours la mettre.

#### `light-dark()` — valeur conditionnelle

```css
:root {
  color-scheme: light dark;
  --ml-color-bg-app:  light-dark(#faf3e0, #1a1815);
  --ml-color-text:    light-dark(#1f1f1f, #f3eada);
  --ml-color-surface: light-dark(#ffffff, #25201c);
  --ml-color-border:  light-dark(#e5d4b1, #3a3530);
}
```

Pattern complet : redéfinir **uniquement les tokens** en `light-dark()`, le reste du CSS reste identique (il consomme `var(--ml-color-bg-app)` sans savoir si on est light ou dark).

Bascule automatique selon `prefers-color-scheme` du système d'exploitation. Pas de `@media`, pas de duplication.

#### Bascule manuelle (toggle bouton)

Pour un theme switcher contrôlé utilisateur (au-delà de la préférence OS) :

```css
:root { color-scheme: light dark; }

/* Override via attribut data sur <html> */
:root[data-theme="dark"]  { color-scheme: dark; }
:root[data-theme="light"] { color-scheme: light; }
```

```javascript
// 3 lignes JS — pas besoin de framework
document.documentElement.dataset.theme = 'dark';
```

Le navigateur recompose `light-dark()` automatiquement → toute l'app bascule sans rechargement.

#### Cible Baseline

- `color-scheme` : **Widely available** depuis 2022. ✅.
- `light-dark()` : **Widely available** depuis fin 2024 (Chrome/Edge 123+, Safari 17.5+, Firefox 120+). Vérifier au jour J pour cible client conservateur, mais OK Dymasco interne.

> 💡 **Si cible client conservateur** : fallback via `@media (prefers-color-scheme: dark) { :root { --ml-color-bg-app: #1a1815; ... } }` (cf. Module 8). Plus verbeux mais widely supporté depuis 2020.

### 3. `oklch()` — citation, pas central (2 min)

Pour les curieux et les rares cas où vous **construisez** une palette from scratch (pas reçue d'un client) :

```css
oklch(L C H)
/* L = lightness perceptive (luminosité) 0-100%
   C = chroma (saturation ou intensité) 0 - ~0.4
   H = hue 0-360° (teinte) */

--ml-color-brand-500: oklch(58% 0.18 35);   /* tomate */
```

Avantages **quand on construit** :

- Échelle "claire → foncée" cohérente en faisant varier L seul
- Famille de teintes harmonieuses en faisant varier H, L+C fixes
- Accès au gamut P3 (écrans modernes)

Outil : [oklch.com](https://oklch.com) — picker visuel hex ↔ oklch.

**Chez Dymasco** : votre charte client arrive en hex. La convertir en oklch n'apporte rien si `color-mix(in oklab, …)` fait déjà le boulot de dérivation. À garder sous le coude pour les rares missions où vous **définissez** la palette (rebrand from scratch, projet pilote interne).

#### Cible Baseline

`oklch()` : **Widely available** depuis 2023. ✅ si besoin.

---

## 🛠️ Exercice fil rouge (22 min)

> 🎨 **Pas de démo séparée** — les couleurs c'est fun, vous testez direct. Tapez le code, ouvrez DevTools, jouez avec les valeurs en live. C'est plus marquant que de me regarder faire.

### 📁 Code de départ

Partez du checkpoint Module 6 : [projet-fil-rouge/checkpoints/06-container-queries/overrides.css](../projet-fil-rouge/checkpoints/06-container-queries/overrides.css)

### Consigne

Reprendre le checkpoint Module 6 et :

1. **Garder les sources hex** (`--ml-color-brand-primary: #c2410c` etc.) — ne pas convertir en oklch.

2. **Créer les variantes dérivées** pour `--ml-color-brand-primary` :
   - `--ml-color-brand-primary-hover` (color-mix, black 10%)
   - `--ml-color-brand-primary-active` (color-mix, black 20%)
   - `--ml-color-brand-primary-disabled` (color-mix, white 50%)

3. **Backgrounds dérivés** pour les statuts :
   - `--ml-color-status-alert-bg` = `color-mix(in oklab, var(--ml-color-status-alert), white 90%)`
   - `--ml-color-status-running-bg` = idem avec running, white 92%
   - **Remplacer** les `color-mix()` inline qui traînent dans le checkpoint M06 par ces tokens.

4. **`color-scheme: light dark`** sur `:root`.

5. **Theme switcher light-dark()** sur 4 tokens minimum :
   - `--ml-color-bg-app`
   - `--ml-color-text`
   - `--ml-color-surface`
   - `--ml-color-border`

6. **Vérifier** dans DevTools → Rendering → `prefers-color-scheme: dark` que la bascule fonctionne sans rien d'autre à modifier.

### 🎮 Tests live à faire vous-même (l'effet wow, en direct)

Une fois le checkpoint en place, jouez dans DevTools :

- **Test 1 — One-line re-harmonisation** : modifier `--ml-color-brand-primary` à la volée sur `:root` (essayez `#1976d2` bleu, `#2e7d32` vert). Hover / active / disabled suivent **tout seuls** grâce à color-mix.
- **Test 2 — Bascule système** : DevTools → Rendering → `prefers-color-scheme: dark`. Toute l'app passe en sombre, **zéro `@media`**.
- **Test 3 — Toggle manuel** : Console DevTools, bascule indépendante de la préférence OS (utile pour un bouton dans l'UI) :

  ```javascript
  document.documentElement.dataset.theme = 'dark';
  document.documentElement.dataset.theme = 'light';
  ```

- **Test 4 — Casser pour comprendre** : retirer `in oklab` d'un `color-mix()`. La couleur disparaît silencieusement (DevTools n'alerte pas). Rappel : **toujours préciser l'espace**.

### Pièges fréquents

- ❌ Mettre `color-mix()` inline partout (oubli de tokenisation) → on perd l'avantage centralisé. **Tokeniser dès qu'un dérivé est utilisé ≥ 2 fois.**
- ❌ Oublier l'espace de mix (`color-mix(<color-1>, <color-2>)` sans `in oklab`) → erreur silencieuse, valeur ignorée. **DevTools n'alerte pas, vérifier visuellement.**
- ❌ `light-dark()` sans `color-scheme: light dark` déclaré → fonctionne, mais les composants UA (scrollbars, inputs) ne suivent pas. Toujours déclarer les deux.
- ❌ Vouloir convertir toute la charte client en oklch "pour faire moderne" → perte de temps. Garder hex sources, mix en oklab.

### Corrigé attendu

Voir `projet-fil-rouge/checkpoints/07-couleurs/overrides.css`.

---

## 🚀 Pour aller plus loin

### Bonus 1 — Échelle complète depuis une teinte (si vous CONSTRUISEZ la palette)

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

→ Toute la gamme depuis **un seul angle de teinte**. Changer `--hue: 220` → tout devient bleu. Cas typique : projet pilote interne où Dymasco choisit la palette, pas le client.

### Bonus 2 — Tokens "primitifs" vs "intentionnels"

Pattern à 2 niveaux :

- **Primitifs** : `--ml-orange-500: #c2410c`, `--ml-cream-100: #fef6e4` (palette brute)
- **Intentionnels** : `--ml-color-brand-primary: var(--ml-orange-500)` (sémantique métier)

Permet de changer la palette sans toucher au code applicatif (les composants consomment uniquement l'intentionnel).

### Bonus 3 — Fallback `light-dark()` pour cible conservateur

```css
:root {
  color-scheme: light dark;
  --ml-color-bg-app: #faf3e0;
}

@supports (color: light-dark(#fff, #000)) {
  :root { --ml-color-bg-app: light-dark(#faf3e0, #1a1815); }
}

@supports not (color: light-dark(#fff, #000)) {
  @media (prefers-color-scheme: dark) {
    :root { --ml-color-bg-app: #1a1815; }
  }
}
```

À utiliser uniquement si la cible client coupe avant fin 2024.

### Ressources

- [MDN — color-mix()](https://developer.mozilla.org/en-US/docs/Web/CSS/color_value/color-mix)
- [MDN — color-scheme](https://developer.mozilla.org/en-US/docs/Web/CSS/color-scheme)
- [MDN — light-dark()](https://developer.mozilla.org/en-US/docs/Web/CSS/color_value/light-dark)
- [MDN — oklch()](https://developer.mozilla.org/en-US/docs/Web/CSS/color_value/oklch)
- [oklch.com](https://oklch.com) — picker visuel (si construction palette)
- Andrey Sitnik, [Better than HSL: Oklch in CSS](https://evilmartians.com/chronicles/oklch-in-css-why-quit-rgb-hsl) — lecture optionnelle

---

## 📝 Récap & checklist

- [ ] Je dérive mes variantes hover/active/disabled via `color-mix()` à partir des sources hex client
- [ ] Je précise **toujours** l'espace de mix : `in oklab` (recommandé) — pas d'oubli silencieux
- [ ] Mes tokens couleurs = **5-10 sources hex** + dérivés `color-mix` tokenisés, pas de hex éparpillé
- [ ] Je déclare `color-scheme: light dark` sur `:root` (1 ligne, gratuit)
- [ ] Je sais faire un theme switcher complet via `light-dark()` sur les tokens
- [ ] Je sais que `oklch()` côté SOURCE = pour construire une palette from scratch (rare chez Dymasco). Côté MIX (`in oklab`) = oui, toujours.

> 🎯 **Mantra du module** : *"Charte client en hex. Dérivés en color-mix. Themes en light-dark. Pas besoin d'oklch."*
