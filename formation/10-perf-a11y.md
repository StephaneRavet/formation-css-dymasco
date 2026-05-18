---
formateur:
  prerequis_formateur:
    - distinguer :focus (souris+clavier) vs :focus-visible (clavier uniquement)
    - connaître par cœur ratios WCAG : 4.5:1 AA / 3:1 large / 7:1 AAA
    - savoir que will-change permanent CONSOMME mémoire GPU (juste avant anim)
    - content-visibility: auto = virtualisation native pour longues listes
    - savoir lancer Lighthouse audit en local
  ressources_demo_a_preparer:
    - checkpoint Module 9 ouvert
    - DevTools onglet Accessibility prêt (contrast ratio en direct)
    - Lighthouse prêt (cible Accessibility 95+ et Performance 95+)
    - WebAIM Contrast Checker en favori
    - check-list mise en prod imprimée à distribuer en clôture
  pitch_ouverture: >
    "Module final. C'est celui qui transforme 'rend bien' en 'LIVRABLE'. À la fin
    vous aurez une check-list imprimable, celle que vous utiliserez avant chaque
    livraison Dymasco. Si vous oubliez tout le reste, gardez ça."
  energie_attendue: posée, conclusive — fin de formation, mode bilan + transmission outils
  duree_cible: 60 min — découpage 25 concepts / 15 démo / 15 exo / 5 récap + distribution check-list
  variantes_timing:
    si_en_retard: zapper bonus prefers-reduced-transparency + skip link
    si_en_avance: lancer Lighthouse en live sur leur projet client le plus récent
  points_a_marteler:
    - ":focus-visible (PAS :focus) = focus ring clavier sans pollution clic souris"
    - "ratios WCAG : 4.5:1 AA texte / 3:1 large ou UI / 7:1 AAA"
    - "will-change UNIQUEMENT au moment de l'animation (hover/focus), pas permanent"
    - "content-visibility: auto = gain massif sur longues listes (journal 1000 lignes)"
    - "ordre du DOM = ordre de tab — flex-direction: row-reverse casse navigation clavier"
    - "check-list mise en prod = LE livrable du module (impression à distribuer)"
  pieges_apprenants:
    - garder :focus au lieu de :focus-visible → outline visible au clic, look amateur
    - will-change permanent sur classe → mémoire GPU consommée pour rien
    - contraste validé en light mais oublié en dark mode
    - content-visibility: auto sur éléments INTERACTIFS → tab clavier saute la zone
    - oublier outline-offset → focus ring colle au bord, illisible sur fond coloré
  questions_probables:
    - q: "Lighthouse score 95+ obligatoire client ?"
      r: viser, pas dogme. Documenter écarts si justifiés.
    - q: "Skip link utile en dashboard atelier ?"
      r: utile si navigation clavier intense. Pattern A11y reconnu.
    - q: "tabindex c'est mal ?"
      r: tabindex="0" OK pour rendre focusable. tabindex>0 = SUPER mal (réordonne).
    - q: "On audite quand pendant le projet ?"
      r: après chaque module ajouté + bilan final avant livraison.
  transition_module_suivant: >
    "Fin de formation. La check-list est votre outil quotidien. Le projet fil rouge
    Mamie Lulu est votre référence pratique. Vous repartez avec : 1 méthode (cible
    + Baseline), 1 architecture (4 couches), 1 boîte à outils CSS moderne, 1 check-list
    de livraison. Bon retour sur vos projets clients."
---

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

### Mesures quantifiées avant/après (fil rouge Mamie Lulu)

À afficher en clôture de formation pour visualiser l'impact total des 10 modules :

| Métrique | Apriso seul (départ) | Fil rouge M10 (arrivée) | Méthode de mesure |
| --- | --- | --- | --- |
| Lighthouse Accessibility | 62 | 98 | `F12 → Lighthouse → Accessibility` |
| Lighthouse Performance | 78 | 96 | `F12 → Lighthouse → Performance` |
| Lighthouse Best Practices | 75 | 100 | idem |
| Nombre de `!important` | 23 (Apriso) | 6 (tous dans `priority`) | grep `!important` |
| Couleurs uniques | 47 | 11 (9 tokens + 2 dérivés) | `F12 → CSS Overview → Capture` |
| Sélecteurs > 3 classes | 41 | 0 | CSS Overview → Specificity |
| Spécificité moyenne (a, b, c) | (0, 3.8, 0.4) | (0, 1.2, 0.1) | CSS Overview |
| Contrast ratios < 4.5:1 | 8 paires | 0 | CSS Overview → Contrast issues |
| Taille du CSS chargé | 11,4 KB (Apriso) + 0 | 11,4 KB (Apriso) + 4,2 KB | DevTools → Network |
| Animations non-GPU (`box-shadow`, `border`) | 6 | 0 | grep `transition:` |
| Cibles tactiles < 44px | 12 | 0 | manuel (DevTools inspect) |

**Lecture pédagogique** :

- Performance n'est pas l'enjeu principal (dashboard statique, parc moderne) — c'est l'A11y et la maintenabilité qui font le gain réel.
- 4,2 KB d'overrides ajoutés vs 11,4 KB d'Apriso = ratio ~37 %. Acceptable pour le gain visuel et a11y obtenu.
- Le `!important` 23 → 6 est le chiffre **à montrer en revue de PR** chez le client. Imparable.

**Comment reproduire le tableau** :

1. Charger `00-base/index.html` (apriso-base.css seul, pas d'overrides) → Lighthouse + CSS Overview = colonne "départ".
2. Charger `checkpoints/10-perf-a11y/index.html` → idem = colonne "arrivée".
3. Capturer les chiffres dans un commentaire en tête du fichier `overrides.css` final. Trace pour la revue client.

### Pièges fréquents

- ❌ Garder `:focus` au lieu de passer à `:focus-visible` → outline visible au clic souris (look amateur).
- ❌ `will-change` permanent → consomme GPU pour rien.
- ❌ Contraste validé en light mode mais oublié en dark mode.
- ❌ `content-visibility: auto` sur des éléments **interactifs** → tab clavier saute la zone non rendue (à valider).
- ❌ `content-visibility: auto` sur du contenu **textuel critique** → la recherche **Ctrl+F** ne trouve pas le texte hors viewport (Safari notamment). Idem pour les lecteurs d'écran qui parcourent le DOM.
- ❌ Oublier `outline-offset` → le focus ring colle au bord, illisible sur fonds colorés.

### Corrigé attendu

Voir `projet-fil-rouge/checkpoints/10-perf-a11y/overrides.css`.

---

## ✅ Check-list de mise en prod (à imprimer)

À cocher avant chaque livraison Dymasco :

### Architecture
- [ ] `.browserslistrc` à la racine, cible client documentée
- [ ] Commentaire "Target / Last reviewed" en tête de chaque `overrides.css`
- [ ] `@layer reset, priority, framework, overrides` déclaré (4 couches — cf. Module 1)
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
- [ ] `forced-colors: active` testé (Windows HC — émulation DevTools + idéalement vrai poste HC)
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
