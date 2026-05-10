# Module 10 — Performance & A11y

> ⏱️ **Durée** : 1h00 — **J2 après-midi, clôture**
> 🧭 **Type** : module check-list mise en prod
> 🎯 **Checkpoint** : `projet-fil-rouge/checkpoints/10-perf-a11y/overrides.css`

---

## 🎯 Objectif (en 1 phrase)

Restaurer un **focus clavier visible**, valider les **contrastes WCAG**, optimiser les **passes navigateur**, et livrer une check-list de mise en prod réutilisable.

---

## ⚙️ Pour qui c'est utile chez Dymasco

C'est le module qui transforme un dashboard "qui rend bien" en dashboard **livrable**. Trois angles :

1. **A11y obligatoire** : un opérateur peut être en environnement bruyant (lecteur d'écran inutile mais clavier oui), ou un superviseur peut avoir des troubles visuels. Le focus clavier doit **être visible**, point.
2. **Contraste WCAG** : norme pro, attendue dans le CDC client. AA = minimum, AAA = bonus.
3. **Performance** : sur tablette opérateur (Galaxy Tab Active5 = SoC modeste), chaque ms compte. Repaints, layout shifts, animations gourmandes → bannis.

Et le piège n°1 dans `apriso-base.css` :

```css
button:focus, select:focus, a:focus { outline: none; }   /* 🚨 RUPTURE A11Y */
```

→ Navigation clavier impossible : aucun feedback visuel quand on tabule. Inacceptable en livraison pro.

---

## 📚 Concepts (25 min)

### 1. `:focus-visible` — le bon focus clavier (8 min)

Avant : `:focus` = à la fois clic souris et tab clavier. Conséquence : on virait l'outline pour ne pas voir le contour bleu au clic — **et on cassait le clavier**.

Aujourd'hui : `:focus-visible` cible **uniquement** le focus issu d'une interaction clavier (ou assistance). Le clic souris ne déclenche pas.

```css
@layer overrides {
  /* Annule le piège Apriso */
  :where(button, select, a, input, textarea):focus { outline: none; }

  /* Restaure un focus VISIBLE clavier */
  :where(button, select, a, input, textarea):focus-visible {
    outline: 2px solid var(--ml-color-brand-primary);
    outline-offset: 2px;
    border-radius: var(--ml-radius-sm);
  }
}
```

→ Souris : pas de outline (look propre). Clavier : outline visible (a11y respectée).

#### Cible Baseline

`:focus-visible` : **Widely available** depuis 2022. ✅.

### 2. Contraste WCAG — les 3 ratios à connaître (5 min)

| Niveau | Ratio | Quoi |
|---|---|---|
| WCAG AA | **4.5:1** | Texte normal |
| WCAG AA Large | **3:1** | Texte ≥ 18px (ou 14px gras) |
| WCAG AAA | **7:1** | Texte normal, niveau pro |
| WCAG AA UI | **3:1** | Composants UI, états (focus ring, bordures actives) |

**Outils** :
- DevTools Chrome → "Inspect" → onglet "Accessibility" → "Contrast ratio" affiché en direct.
- WebAIM Contrast Checker.
- Lighthouse audit.

> 💡 Avec `oklch` (Module 7), augmenter le contraste = **baisser la lightness** sans changer la teinte. Beaucoup plus prévisible qu'en hex.

### 3. Navigation clavier — l'ordre de tabulation (4 min)

Règle : **l'ordre du DOM = l'ordre de tab**. Si on a réorganisé via Grid (`order`, `grid-template-areas`), le tab peut suivre un ordre **différent** du visuel → désorientation.

Tester en passant de tab en tab → vérifier que l'ordre reste cohérent.

```css
/* À éviter — modifie l'ordre visuel sans modifier le DOM */
.apriso-header { display: flex; flex-direction: row-reverse; }   /* 🚨 tab inversé */
```

Préférer changer l'ordre **dans le HTML** ou utiliser `tabindex` avec parcimonie.

### 4. `will-change` — l'arme à utiliser avec précaution (4 min)

Indique au navigateur qu'une propriété **va** être animée → il prépare une couche de composition GPU.

```css
.apriso-machine-card:hover { will-change: transform; }
```

→ Hover déclare "je vais animer transform", le navigateur alloue la couche **avant** que ça arrive.

> ⚠️ **Piège** : `will-change` permanent **consomme la mémoire** GPU. À utiliser :
> - Sur **peu** d'éléments (pas tous les items d'une liste).
> - **Juste avant** l'animation (hover, focus), pas en permanence.
> - Et **retirer** une fois l'animation finie (via JS ou via état).

Cas concret Dymasco : sur une grille de **50 cards**, mettre `will-change: transform` sur `:hover` est OK (un seul élément à la fois). Mettre `will-change: transform` sur la classe `.apriso-machine-card` (toutes) → problème.

### 5. `content-visibility: auto` — virtualisation native (4 min)

Pour une **longue liste** (journal d'événements de plusieurs centaines de lignes), permet au navigateur de skipper le rendu des éléments **hors viewport**.

```css
.apriso-event-log__table tbody tr {
  content-visibility: auto;
  contain-intrinsic-size: 0 36px;   /* hauteur estimée */
}
```

→ Réduction massive du temps de layout pour de grandes listes.

Cible Baseline : **Newly available** (Chrome/Edge 85+, Safari 18+, Firefox 125+). ✅ pour Dymasco, à vérifier sur cible client.

---

## 🔍 Démonstration (15 min)

### Étape 1 — Restaurer le focus clavier (5 min)

Tester avec **Tab** sur le dashboard avant modif → aucun feedback. Catastrophe.

Ajouter le bloc `:focus-visible` (cf. concept §1). Re-tester avec Tab → halo orange Mamie Lulu sur le sélecteur, sur les éléments interactifs. Lisible immédiatement.

### Étape 2 — Audit contraste (4 min)

DevTools → "Inspect" sur le footer time (`color: var(--ml-color-text-muted)` sur `--ml-color-bg-cream`). Lire le ratio. Si < 4.5:1 → assombrir `--ml-color-text-muted` jusqu'à passer.

Lighthouse → Audit Accessibility. Cible : 100.

### Étape 3 — Performance check (3 min)

DevTools → Performance → enregistrer 5s avec hover sur 5 cards.

Vérifier :
- Pas de "Layout" rouge → seul `transform` est animé.
- Pas de "Recalculate Style" massif → `:has()` modéré.
- FPS > 50.

### Étape 4 — `content-visibility` sur le journal (3 min)

Ajouter `content-visibility: auto` sur les `<tr>`. Si le journal est long (simuler en dupliquant les lignes), gain visible immédiat.

---

## 🛠️ Exercice fil rouge (15 min)

### Consigne

Reprendre le checkpoint Module 9 et :

1. **`:focus-visible`** : annuler le `:focus { outline: none }` du framework, ajouter un focus ring visible (couleur brand, offset 2px, radius).

2. **Contraste** :
   - Vérifier `--ml-color-text-muted` sur `--ml-color-bg-cream` → assombrir si < 4.5:1.
   - Vérifier le badge alert (texte sur fond) → idem.
   - Documenter les ratios mesurés en commentaire.

3. **`content-visibility`** : appliquer sur les `<tr>` du journal d'événements pour préparer un volume important.

4. **`will-change` ciblé** : sur `.apriso-machine-card:hover`, ajouter `will-change: transform`. Pas en permanence.

5. **Cleanup** : passer en revue le CSS, retirer toute trace de :
   - `transition: all`
   - `!important` non documenté (le seul autorisé : reduced-motion)
   - Hardcode hex (tout doit passer par `--ml-color-*`)

### Tests à faire

- **Tab clavier** : naviguer dans tous les contrôles, vérifier que **chaque** élément focusable a un ring visible.
- **Lighthouse Accessibility** : score cible 95+.
- **Lighthouse Performance** : score cible 95+ (le dashboard est statique, doit être facile).
- **DevTools Contrast** : valider toutes les paires texte/fond utilisées.

### Pièges fréquents

- ❌ Garder `:focus` au lieu de passer à `:focus-visible` → outline visible au clic souris (look amateur).
- ❌ `will-change` permanent → consomme GPU pour rien.
- ❌ Contraste validé en light mode mais oublié en dark mode.
- ❌ `content-visibility: auto` sur des éléments **interactifs** → tab clavier saute la zone non rendue (à valider).
- ❌ Oublier `outline-offset` → le focus ring colle au bord, illisible sur fonds colorés.

### Corrigé attendu

Voir `projet-fil-rouge/checkpoints/10-perf-a11y/overrides.css`.

---

## ✅ Check-list de mise en prod (à imprimer)

À cocher avant chaque livraison Dymasco :

### Architecture
- [ ] `.browserslistrc` à la racine, cible client documentée
- [ ] Commentaire "Target / Last reviewed" en tête de chaque `overrides.css`
- [ ] `@layer reset, framework, overrides` déclaré
- [ ] Aucun `!important` hors reduced-motion

### Tokens
- [ ] Toutes les couleurs en `oklch()` ou via `--ml-color-*`
- [ ] Aucun hex hardcodé dans le code applicatif
- [ ] Variantes hover/active/disabled dérivées via `color-mix()`

### Layout
- [ ] Largeurs en `clamp()` / `min()` / `max()` ou `min-content` / `max-content`
- [ ] Plus aucune `width: Npx` figée (sauf bordures)
- [ ] `dvh` plutôt que `vh` pour les hauteurs plein écran

### Composants
- [ ] Cards adaptatives via Container Queries si réutilisables
- [ ] Triptyque overflow appliqué partout : ellipsis, line-clamp, overflow-wrap
- [ ] `min-width: 0` sur les ancêtres flex de textes ellipsés

### A11y
- [ ] `:focus-visible` sur **tous** les éléments interactifs
- [ ] Contraste **AA minimum** sur tous les couples texte/fond (light ET dark)
- [ ] `aria-hidden="true"` sur les pictos décoratifs
- [ ] Cibles tactiles ≥ 44×44 px en mode tactile
- [ ] Ordre du DOM = ordre visuel (tab cohérent)

### Performance
- [ ] Aucune `transition: all`
- [ ] Animations sur `transform` / `opacity` uniquement
- [ ] `will-change` uniquement sur hover/focus, pas permanent
- [ ] `content-visibility: auto` sur les longues listes

### Modes utilisateur
- [ ] Mode sombre testé
- [ ] `prefers-reduced-motion` testé
- [ ] `prefers-contrast` testé
- [ ] Print preview testé
- [ ] Tactile coarse testé

### Tests finaux
- [ ] Lighthouse Accessibility ≥ 95
- [ ] Lighthouse Performance ≥ 95
- [ ] Test visuel sur 3 résolutions (tablette / PC / mural)
- [ ] Test tab clavier complet
- [ ] Test avec un nom de machine **très long**

---

## 🚀 Pour aller plus loin

### Bonus 1 — `prefers-reduced-transparency`

```css
@media (prefers-reduced-transparency: reduce) {
  .modal-backdrop { background: solid; }
}
```

### Bonus 2 — Skip link

```html
<a href="#main" class="apriso-skip-link">Aller au contenu</a>
```

```css
.apriso-skip-link {
  position: absolute;
  inline-size: 1px;
  block-size: 1px;
  overflow: hidden;
  clip-path: inset(50%);
}
.apriso-skip-link:focus-visible {
  position: fixed;
  inset-block-start: 0;
  inline-size: auto;
  block-size: auto;
  padding: var(--ml-spacing-3);
  background: var(--ml-color-brand-primary);
  color: white;
  z-index: 100;
}
```

→ Pattern A11y "skip to content" pour les utilisateurs clavier.

### Ressources

- [WebAIM — Contrast Checker](https://webaim.org/resources/contrastchecker/)
- [MDN — :focus-visible](https://developer.mozilla.org/en-US/docs/Web/CSS/:focus-visible)
- [MDN — content-visibility](https://developer.mozilla.org/en-US/docs/Web/CSS/content-visibility)
- [MDN — will-change](https://developer.mozilla.org/en-US/docs/Web/CSS/will-change)
- [WCAG 2.1 — Quick Reference](https://www.w3.org/WAI/WCAG21/quickref/)
- [Lighthouse — Accessibility audits](https://developer.chrome.com/docs/lighthouse/accessibility/)

---

## 📝 Récap & checklist

- [ ] J'utilise `:focus-visible` au lieu de `:focus` pour les rings clavier
- [ ] J'audite mes contrastes via DevTools (4.5:1 mini)
- [ ] J'utilise `will-change` uniquement au moment de l'animation
- [ ] J'utilise `content-visibility: auto` sur mes longues listes
- [ ] J'ai imprimé / sauvegardé la check-list de mise en prod
- [ ] Je passe Lighthouse A11y et Perf en local avant livraison

> 🎯 **Mantra du module** : *"Si je ne peux pas faire le tour du dashboard au clavier en 30 secondes, ce n'est pas livrable."*
