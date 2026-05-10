# Module 0 — Cible navigateur & arbitrage

> ⏱️ **Durée** : 1h00 — **J1 matin, ouverture**
> 🧭 **Type** : module méthodologique (pas d'exercice dashboard, pas de checkpoint `overrides.css`)

---

## 🎯 Objectif (en 1 phrase)

Savoir **arbitrer** quelle feature CSS moderne on peut utiliser pour un client donné, en s'appuyant sur des sources factuelles (`caniuse`, **Baseline**, `browserslist`) plutôt que sur l'intuition ou l'habitude.

---

## ⚙️ Pour qui c'est utile chez Dymasco

Vous livrez du CSS de surcharge à des clients qui ont chacun **leur propre parc navigateur**. Côté infra Dymasco : Edge/Chromium ≤3 ans, **tout passe**. Côté clients : ça dépend du CDC.

Cas concrets :
- Client A (agroalimentaire, parc PC d'atelier renouvelé en 2024) → Chromium récent, vous pouvez tout utiliser.
- Client B (industrie lourde, postes Windows 10 + Edge non mis à jour) → vérifier feature par feature.
- Client C (sous-traitants, BYOD tablette) → cible mouvante, prévoir des fallbacks.

**Le bon réflexe** : à chaque début de mission, fixer la cible **par écrit**, l'inscrire dans le `browserslist`, et utiliser ça comme référence d'arbitrage tout au long de l'intégration.

---

## 📚 Concepts (20 min)

### 1. La cible navigateur, une **décision projet**, pas une habitude

Trois questions à poser au début de chaque intégration :

1. **Quel parc** est en face ? (versions, OS, type de poste)
2. **Quelle politique de mise à jour** ? (auto, gérée IT, jamais)
3. **Quelle durée de vie** pour la livraison ? (6 mois ? 3 ans ?)

Ces réponses se traduisent en une **cible navigateur écrite** dans le projet. Pas dans une tête.

### 2. `browserslist` — la source unique

`browserslist` est le standard de fait. Lu par PostCSS, Autoprefixer, esbuild, Vite, etc. Même sans pipeline de build chez Dymasco, c'est un excellent **document d'arbitrage** : on l'écrit dans un `.browserslistrc` à la racine du projet d'override, et il sert de référence partagée.

Syntaxe utile :

```
# .browserslistrc — exemple cible "client industriel récent"
last 2 Chrome versions
last 2 Edge versions
not dead
```

> 📦 **`not dead` — définition**
>
> Filtre qui **exclut les navigateurs morts** : sans mise à jour officielle depuis **24 mois** ET part de marché globale **< 0,5 %**.
>
> Concrètement, retire de la cible : IE 10/11, IE Mobile, BlackBerry Browser, vieilles Opera Mini, KaiOS Browser, anciennes Samsung Internet.
>
> Liste maintenue par le projet `browserslist` via `caniuse-lite`. Pour la voir :
> ```bash
> npx browserslist "dead"
> ```
>
> **Pourquoi le garder** : `last 2 versions` peut embarquer la dernière version d'un navigateur fossile (ex : IE 11 est la "dernière version" d'IE). `not dead` agit comme filet de sécurité — combiné avec une autre query, jamais seul.
>
> Pour Dymasco : effet quasi nul (cible Edge/Chromium récent), mais coût zéro → bonne hygiène, on le garde.

Ou en version "client conservateur" :

```
Edge >= 110
Chrome >= 110
not dead
```

À tester en CLI :

```bash
npx browserslist "last 2 Chrome versions, last 2 Edge versions"
```

→ liste les versions exactes ciblées **aujourd'hui**.

### 3. **Baseline** — le nouveau langage commun

Initiative lancée par le WebDX Community Group (Google, Mozilla, Apple, Microsoft) pour offrir une vue unifiée de l'état du Web. Baseline catégorise chaque feature selon son support :

- **Widely available** : supportée par les **4 navigateurs majeurs** (Chrome, Edge, Firefox, Safari) depuis ≥ 30 mois (≈ 2,5 ans). Utilisable sans risque.
- **Newly available** : supportée par les 4 majeurs, mais depuis moins de 30 mois. À vérifier selon votre cible.
- **Limited availability** : pas encore supportée par les 4 majeurs.

> 🔗 **Exemple concret** : ouvrez la fiche MDN du sélecteur [**:has()**](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Selectors/:has). Tout en haut, sous le titre, le badge **Baseline** indique son statut (passé **Widely available** fin 2023). Ce badge est désormais présent sur **chaque** fiche MDN d'une feature CSS, JS ou HTML — c'est votre référence n°1 d'arbitrage.

Lecture rapide :

| Statut Baseline | Décision par défaut |
|---|---|
| Widely available | Utiliser sans réserve |
| Newly available | Vérifier la cible client, sinon `@supports` |
| Limited | Fallback obligatoire ou refuser |

À consulter sur [web.dev/baseline](https://web.dev/baseline) ou directement sur les fiches MDN (badge Baseline en haut).

### 4. `caniuse` — granularité fine

Quand `Baseline` ne suffit pas (ex : on cible une version Edge précise), `caniuse.com` donne la matrice détaillée.

Réflexe : ouvrir `caniuse.com/?search=container+queries` plutôt que demander à un LLM.

### 5. `@supports` — feature detection runtime

`@supports` est une règle CSS qui n'applique son bloc que si le navigateur reconnaît la paire `propriété: valeur` testée. Équivalent d'un `if (navigator.knows(feature))`, mais en CSS pur.

```css
/* fallback par défaut */
.card { display: block; }

/* progressive enhancement */
@supports (container-type: inline-size) {
  .card { container-type: inline-size; }
}
```

Sans `@supports`, un vieux navigateur ignore silencieusement une propriété qu'il ne comprend pas — mais tu ne peux pas écrire de **fallback différent**. Avec `@supports`, tu sépares proprement les deux chemins :

- **fallback** par défaut → tous les navs.
- **upgrade** dans le bloc `@supports` → seulement les navs récents.

Quand on doit livrer pour une cible mixte, `@supports` permet d'écrire un **chemin moderne** + un **fallback**.



Variante avec négation :

```css
@supports not (color: oklch(50% 0.1 200)) {
  /* fallback hex pour vieux navigateurs */
  .button { background: #ff6b35; }
}
```

> ⚠️ **Piège** : `@supports` teste la **syntaxe**, pas le **bon fonctionnement**. `@supports (display: grid)` est `true` même sur un navigateur qui implémente Grid à moitié. C'est rare aujourd'hui, mais à connaître.

### 6. Lire un CDC client — extraire la cible

Un CDC ne dit jamais "Chrome ≥ 115". Il dit :
- *"Compatible avec les navigateurs en cours de support éditeur"*
- *"Postes Windows 10 d'atelier"*
- *"Tablettes Android sous Chrome, version stable"*

→ Votre travail : **traduire** en `browserslist`. Et **valider par écrit** avec le client avant de coder.

---

## 🔍 Démonstration (15 min)

**Scénario** : on doit choisir si on utilise `@layer` (Module 1) sur le projet Mamie Lulu.

Démarche live :

1. **Vérifier Baseline** : [developer.mozilla.org/en-US/docs/Web/CSS/@layer](https://developer.mozilla.org/en-US/docs/Web/CSS/@layer) → badge "Widely available".
2. **Vérifier [caniuse](https://caniuse.com/mdn-css_at-rules_layer)** : Chrome 99+, Edge 99+, Safari 15.4+, Firefox 97+ → tous depuis début 2022.
3. **Croiser avec la cible Dymasco** : Edge/Chromium ≤ 3 ans → ✅ on a marge confortable.
4. **Décision écrite** : ajouter une note dans `overrides.css` :
   ```css
   /* Target: Edge/Chromium <= 3 ans (Baseline widely available features OK) */
   ```

Contre-exemple, même démarche pour `@scope` :

1. [Baseline](https://developer.mozilla.org/fr/docs/Web/CSS/Reference/At-rules/@scope) → "Limited availability" (Safari récent uniquement).
2. [caniuse](https://caniuse.com/css-cascade-scope) → pas dans Firefox stable.
3. Décision : **non utilisé** dans la formation. C'est explicitement dans les modules **non couverts** (cf. CONTEXT.md).

**Bilan** : la décision se prend en **3 minutes**, sources à l'appui.

---

## 🛠️ Cas concrets — Lecture de 3 CDC (20 min)

### Consigne

Pour chacun des 3 CDC ci-dessous : produire **(a)** un `.browserslistrc` argumenté et **(b)** un tableau "feature → décision" pour 5 features de la formation : `@layer`, `:has()`, Container Queries, `oklch()`, Subgrid.

### CDC 1 — "Pâtes Mamie Lulu" (notre fil rouge)

> Extrait : *"Le module Forge sera déployé sur les postes superviseurs (PC industriels Windows 11, Edge installé par l'IT, mises à jour automatiques) et sur 4 tablettes opérateurs Samsung Galaxy Tab Active5 sous Android 14 (Chrome stable). Pas d'usage public web."*

Décision : cible large + récente, **toutes les features de la formation passent**.

### CDC 2 — Industriel automobile fictif

> Extrait : *"Le poste cible est un PC d'atelier sous Windows 10 LTSC, Edge non mis à jour automatiquement, version figée par DSI. Validation IT requise pour toute mise à jour. Durée de vie attendue de la solution : 5 ans."*

Décision : cible figée à une version Edge précise (à demander). `browserslist` du type `Edge 109` (dernière version supportée sur certains OS legacy). `@layer` ✅, `:has()` à vérifier sur la version exacte, Container Queries probablement ✅, `oklch` ✅, Subgrid à vérifier.

### CDC 3 — ETI agroalimentaire avec sous-traitants BYOD

> Extrait : *"Le tableau de bord sera consulté par 12 collaborateurs internes (parc Chrome récent géré IT) et par environ 30 partenaires sous-traitants utilisant leur propre matériel. Aucune contrainte navigateur n'est imposée aux partenaires."*

Décision : cible large et **conservatrice** (le maillon faible des sous-traitants pilote). Probablement `last 4 Chrome versions, last 4 Edge versions, last 2 Firefox versions, last 2 Safari versions, not dead`. Pour les features récentes : `@supports` systématique.

---

## 🚀 Pour aller plus loin

### Bonus : `@supports selector()`

Tester la disponibilité d'un sélecteur moderne :

```css
@supports selector(:has(*)) {
  .alert-row:has(.status--critical) { background: #fee; }
}
```

Plus précis que `@supports (display: grid)` quand on cible un sélecteur récent.

### Ressources

- [web.dev/baseline](https://web.dev/baseline) — entrée principale
- [MDN — Baseline](https://developer.mozilla.org/en-US/docs/Glossary/Baseline) — définition et glossaire
- [MDN — Sélecteur :has()](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Selectors/:has) — exemple de feature Baseline
- [caniuse.com](https://caniuse.com) — matrice détaillée
- [browsersl.ist](https://browsersl.ist) — éditeur visuel de queries `browserslist`
- [MDN — @supports](https://developer.mozilla.org/en-US/docs/Web/CSS/@supports)
- Badge Baseline sur **chaque** fiche MDN CSS (en haut de page)

### Dans votre projet Dymasco demain

1. Ajouter un `.browserslistrc` à la racine de chaque projet d'override.
2. Mettre une **ligne de commentaire en tête** de chaque `overrides.css` :
   ```css
   /* Target: <browserslist query> — last reviewed: YYYY-MM-DD */
   ```
3. Au moindre doute sur une feature : `Baseline` → `caniuse` → décision écrite.

---

## 📝 Récap & checklist

- [ ] Je sais énoncer la **cible navigateur** d'un projet en `browserslist`
- [ ] Je sais lire le badge **Baseline** sur MDN
- [ ] Je sais croiser **Baseline** + **caniuse** + **CDC client** pour décider
- [ ] J'écris `@supports` quand la cible est mixte
- [ ] Je documente la cible **dans le CSS** (commentaire en tête)
- [ ] Je sais que `@scope`, CSS Anchor, View Transitions, `if()` sont **hors périmètre** de cette formation (trop récents ou hors cas d'usage Dymasco)

> 🎯 **Mantra du module** : *"On ne dit plus 'je crois que ça passe'. On dit 'Baseline widely / cible client OK'."*
