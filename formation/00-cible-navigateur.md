---
formateur:
  prerequis_formateur:
    - savoir naviguer caniuse.com en live (URL directe par feature)
    - avoir une fiche MDN ouverte d'avance pour montrer le badge Baseline
    - connaître la définition exacte de "not dead" (maj 24 mois + > 0.5% part de marché)
  ressources_demo_a_preparer:
    - onglet MDN :has() pour montrer badge Baseline
    - onglet caniuse @layer ouvert
    - terminal avec npx browserslist prêt
  pitch_ouverture: >
    "Vous êtes seniors, vous savez écrire du CSS. Ce qu'on va faire ensemble, c'est
    arbitrer : qu'est-ce qu'on peut s'autoriser chez tel client, et comment on le
    justifie sans répondre 'je crois que ça passe'."
  energie_attendue: haute — c'est l'ouverture, on pose le ton "seniors qui décident"
  duree_cible: 60 min — découpage 5 / 20 / 15 / 20
  variantes_timing:
    si_en_retard: couper CDC 2 et 3, garder seulement CDC 1 Mamie Lulu (gain 12 min)
    si_en_avance: faire ouvrir caniuse en live sur une feature au choix du groupe
  points_a_marteler:
    - Baseline = la nouvelle référence n°1 (badge visible en haut de chaque fiche MDN)
    - browserslist = document d'arbitrage écrit, pas un outil de build
    - "@supports teste la syntaxe, pas le bon fonctionnement"
    - décision = 3 sources (Baseline + caniuse + CDC) + écrit dans le code
  pieges_apprenants:
    - confondre cible Dymasco (Edge récent, tout passe) et cible CLIENT (variable)
    - vouloir un fallback "au cas où" même quand Baseline widely OK → surcoût inutile
    - lire caniuse uniquement (manquer le statut Baseline)
  questions_probables:
    - q: "Et si le client n'a pas de CDC technique précis ?"
      r: Tu écris la cible TOI, tu la fais valider par mail. Ça devient la référence.
    - q: "browserslist sans pipeline de build, ça sert à quoi ?"
      r: Document d'arbitrage partagé équipe + commentaire en tête overrides.css.
    - q: "Baseline vs caniuse, lequel je consulte ?"
      r: Baseline d'abord (verdict rapide), caniuse si besoin de granularité version.
    - q: "Pourquoi @scope est exclu ?"
      r: Limited availability (Safari récent only). Hors périmètre formation.
  transition_module_suivant: >
    "Maintenant qu'on sait DÉCIDER ce qu'on peut utiliser, on attaque le sujet
    qui fait perdre le plus de temps en intégration : la cascade et la spécificité.
    Direction le module 1."
---

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

> 🏭 **Précision Apriso d'emblée** : le runtime DELMIA Apriso (Process Builder + Screen Manager, IIS) **ne lit pas** `.browserslistrc`. Pas de pipeline Node embarqué côté serveur. Le CSS saisi dans le HTML Layout Editor part tel quel au navigateur opérateur.
>
> **Chez Dymasco, `browserslist` n'a qu'un seul usage : document d'arbitrage humain.** Pas de build externe prévu côté équipe. Fichier dans le repo + commentaire en tête d'`overrides.css` → trace écrite, revue client, alignement équipe. Apriso s'en fout, c'est pour les humains. Tant que ce choix tient, on saute le pipeline.

`browserslist` est le standard de fait dans l'écosystème front (lu par PostCSS, Autoprefixer, esbuild, Vite…). Côté Dymasco, on **emprunte uniquement le format** comme convention d'écriture : on l'écrit dans un `.browserslistrc` à la racine du projet d'override, et il sert de référence partagée — sans outil de build derrière.

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

> 💡 Pour tester/valider une query sans installer Node : [browsersl.ist](https://browsersl.ist) — éditeur visuel en ligne. Suffit pour l'usage "document d'arbitrage" Dymasco.

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

## 🛠️ Cas concrets — 3 CDC à arbitrer (35 min)

### Consigne (commune aux 3 exercices)

Pour chaque CDC : produire **(a)** un `.browserslistrc` argumenté (versions fixes de préférence) et **(b)** un tableau "feature → décision" pour les 5 features clés de la formation : `@layer`, `:has()`, `@container`, `oklch()`, Subgrid.

**Découpage** :

- **Exercice 1** — démonstration faite **ensemble** au tableau (15 min).
- **Exercices 2 et 3** — chaque apprenant seul **5 min** sur sa feuille, puis correction **dérouée en live** (~5 min) — même format que l'exercice 1.

---

### 🔍 Exercice 1 — CDC "Pâtes Mamie Lulu" (démonstration ensemble, 15 min)

> *"Le module Forge sera déployé sur les postes superviseurs (PC industriels Windows 11, Edge installé par l'IT, mises à jour automatiques) et sur 4 tablettes opérateurs Samsung Galaxy Tab Active5 sous Android 14 (Chrome stable). Pas d'usage public web."*

#### Étape 1 — Extraire la cible du CDC

| Élément CDC | Traduction technique |
| --- | --- |
| Postes superviseurs Windows 11, Edge installé par l'IT, MAJ auto | **Edge récent**, retard max ~6 mois sur stable |
| 4 tablettes Galaxy Tab Active5, Android 14, Chrome stable | **Chrome Android récent** (Play Store à jour) |
| Pas d'usage public web | **Pas de Firefox ni Safari** dans le parc → on ne les porte pas |
| Pas de mention de durée de vie | À clarifier par mail client — par défaut 24 mois |

#### Étape 2 — Le `.browserslistrc` argumenté

```text
# .browserslistrc — projet Mamie Lulu / Forge override
# Cible parc : Edge superviseurs (Win11, auto-update IT)
#            + Chrome Android (Galaxy Tab Active5, Android 14)
# Hors cible : Firefox, Safari (intranet industriel, pas de public web)
# Revue : 2026-05-17 — prochaine relecture dans 12 mois ou si changement de parc

Edge >= 120
ChromeAndroid >= 120
not dead
```

Pourquoi des **versions fixes** plutôt que `last 2 versions` :

- `last 2 Edge versions` couvre ~8 semaines en arrière — trop court pour audit client et pour engagement contractuel ("on garantit jusqu'à quoi exactement ?").
- Une version plancher chiffrée est **vérifiable** : le client peut interroger sa DSI et confirmer "tous nos postes sont ≥ Edge 120". `last 2 versions` est mouvant, impossible à valider.
- Choix du plancher = max(version requise par les 5 features de la matrice) + marge. Subgrid demande Edge 117 → on prend **120** (3 versions de marge ≈ 3 mois de tolérance pour postes en retard).
- Edge ≥ 120 et Chrome Android ≥ 120 sont sortis fin 2023 → > 2 ans de recul à date, parfaitement réalistes pour un parc auto-update IT en 2026.
- `not dead` → filet de sécurité (cf. encadré §2).
- **Pas de Firefox / Safari** → aucune présence dans le parc, on ne gonfle pas inutilement la matrice de support.

À valider en CLI avant envoi client :

```bash
npx browserslist "Edge >= 120, ChromeAndroid >= 120, not dead"
```

#### Étape 3 — Tableau de décision "feature → décision"

| Feature | Baseline | Première version OK | Statut cible Mamie Lulu | Décision | Note livraison |
| --- | --- | --- | --- | --- | --- |
| `@layer` (Module 1) | [**Widely** depuis mars 2022](https://developer.mozilla.org/en-US/docs/Web/CSS/@layer) | [Edge 99 / Chrome 99](https://caniuse.com/css-cascade-layers) | Plancher 120 ≫ 99 (+21) | ✅ Utiliser directement | Pierre angulaire de l'archi `overrides.css` |
| `:has()` (Module 5) | [**Widely** depuis déc 2023](https://developer.mozilla.org/en-US/docs/Web/CSS/:has) | [Edge 105 / Chrome 105](https://caniuse.com/css-has) | Plancher 120 ≫ 105 (+15) | ✅ Utiliser directement | Styling parent-driven (lignes critiques) |
| `@container` (Module 6) | [**Widely** depuis févr 2023](https://developer.mozilla.org/en-US/docs/Web/CSS/@container) | [Edge 105 / Chrome 105](https://caniuse.com/css-container-queries) | Plancher 120 ≫ 105 (+15) | ✅ Utiliser directement | Préférer aux media queries pour composants dashboard |
| `oklch()` (Module 7) | [**Widely** depuis mai 2023](https://developer.mozilla.org/en-US/docs/Web/CSS/color_value/oklch) | [Edge 111 / Chrome 111](https://caniuse.com/css-oklch-oklab) | Plancher 120 ≫ 111 (+9) | ✅ Utiliser directement | Tokens couleur en `oklch` — gradients perceptuels propres |
| Subgrid (Module 4) | [**Widely** depuis sept 2023](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_grid_layout/Subgrid) | [Edge 117 / Chrome 117](https://caniuse.com/css-subgrid) | Plancher 120 ≫ 117 (+3) | ✅ Utiliser directement | Usage ciblé : tableaux 2D alignés, ne pas en abuser |

#### Étape 4 — Bilan et trace écrite

Cible Edge + Chrome Android récents avec auto-update → **les 5 features passent sans `@supports`**. La décision tient en 3 minutes, sources à l'appui.

On documente le verdict en tête d'`overrides.css` :

```css
/* Target: Edge >= 120, ChromeAndroid >= 120, not dead
   Scope: Forge / Mamie Lulu — pas de public web
   Last reviewed: 2026-05-17 — toutes features Baseline widely OK */
```

> 💡 **Mamie Lulu est le cas facile**. Sur les exercices 2 et 3, le tableau aura des `❌` et des lignes "`@supports` obligatoire" ou "refuser la feature". L'objectif : **détecter quand le ✅ n'est plus automatique**.

---

### ✏️ Exercice 2 — Industriel automobile fictif (5 min seul + correction live)

> **Énoncé** : *"Le poste cible est un PC d'atelier sous Windows 10 LTSC, Edge non mis à jour automatiquement, version figée par DSI. Validation IT requise pour toute mise à jour. Durée de vie attendue de la solution : 5 ans."*

**5 min** pour produire votre propre `.browserslistrc` + tableau de décision. Puis on déroule la correction ensemble.

#### 🔑 Correction — Exercice 2 (à dérouler après les 5 min)

##### Étape 1 — Extraction (exo 2)

| Élément CDC | Traduction technique |
| --- | --- |
| Win 10 LTSC, Edge non mis à jour auto, version figée par DSI | **Edge plancher inconnu** → **à demander à la DSI client** avant validation |
| Validation IT pour toute MAJ | Le plancher d'aujourd'hui sera quasi le plancher dans 1 an |
| Durée de vie 5 ans | On s'engage jusqu'en 2031 sur le parc **actuel** — pas sur le futur |
| Aucune mention d'Android / tablettes / public | Edge desktop **uniquement** |

**Action obligatoire avant code** : mail DSI client pour obtenir la version Edge exacte en parc. Sans réponse, on prend un plancher conservateur (hypothèse de travail).

##### Étape 2 — `.browserslistrc` (exo 2)

```text
# .browserslistrc — projet Automobile / Atelier override
# Cible parc : Edge desktop figé DSI (Win 10 LTSC, MAJ manuelle)
# Plancher HYPOTHÈSE — à confirmer par mail DSI client avant validation finale
# Hors cible : Chrome, Firefox, Safari, mobile
# Revue : 2026-05-17 — relecture obligatoire si DSI confirme version réelle

Edge >= 110
not dead
```

Justification :

- **Plancher 110 = hypothèse conservatrice** sans info DSI. Edge 110 sorti févr 2023, plausible pour un parc LTSC mis à jour il y a 1-2 ans.
- Pas de `last N versions` → cible **figée**, le `last` mouvant n'a aucun sens ici.
- Si DSI répond "Edge 130" → on remonte le plancher (bonus pour nous). Si DSI répond "Edge 95" → on redescend et on rouvre le tableau de décision.

##### Étape 3 — Tableau de décision (exo 2, plancher Edge 110)

| Feature | Première version OK | Statut cible (plancher 110) | Décision |
| --- | --- | --- | --- |
| `@layer` | Edge 99 | 110 ≫ 99 (+11) | ✅ Utiliser directement |
| `:has()` | Edge 105 | 110 ≫ 105 (+5, marge fine) | ⚠️ Utiliser, mais **prévoir `@supports selector(:has(*))`** pour les blocs critiques |
| `@container` | Edge 105 | 110 ≫ 105 (+5) | ✅ Utiliser, marge correcte |
| `oklch()` | Edge 111 | 110 < 111 (−1) | ❌ **Fallback hex/rgb obligatoire** via `@supports (color: oklch(0% 0 0))` |
| Subgrid | Edge 117 | 110 < 117 (−7) | ❌ **Refuser ou fallback grid simple** — pas de bricolage |

##### Étape 4 — Bilan (exo 2)

Cible figée + 5 ans de durée de vie = **2 features sortent du périmètre Baseline** (`oklch`, Subgrid). Sur 5 ans, le parc ne bougera quasiment pas → ne pas s'engager sur ce qui ne passe pas aujourd'hui.

```css
/* Target: Edge >= 110 (hypothèse — à confirmer DSI client) — Win 10 LTSC figé
   Scope: Automobile / Atelier
   Last reviewed: 2026-05-17 — oklch + Subgrid HORS périmètre cible
   Engagement contractuel : 5 ans → revue version Edge tous les 12 mois */
```

---

### ✏️ Exercice 3 — ETI agroalimentaire avec sous-traitants BYOD (5 min seul + correction live)

> **Énoncé** : *"Le tableau de bord sera consulté par 12 collaborateurs internes (parc Chrome récent géré IT) et par environ 30 partenaires sous-traitants utilisant leur propre matériel. Aucune contrainte navigateur n'est imposée aux partenaires."*

**5 min** pour produire votre propre `.browserslistrc` + tableau de décision. Puis correction ensemble.

#### 🔑 Correction — Exercice 3 (à dérouler après les 5 min)

##### Étape 1 — Extraction (exo 3)

| Élément CDC | Traduction technique |
| --- | --- |
| 12 internes Chrome récent géré IT | Chrome desktop **récent**, fenêtre courte (auto-update) |
| 30 sous-traitants BYOD, aucune contrainte | **Tout est possible** : Chrome / Edge / Firefox / Safari (desktop + mobile), versions inconnues |
| Aucune contrainte navigateur imposée aux partenaires | **Maillon faible pilote** la cible — on doit s'aligner sur le pire plausible |
| Pas de mention de durée de vie | À clarifier par mail — par défaut 24 mois |

**Action obligatoire avant code** : mail au client pour faire **valider par écrit** une clause type *"navigateurs supportés : versions stables sorties dans les 24 derniers mois"*. Sans cette clause, n'importe quel sous-traitant peut ouvrir le dashboard sous Safari 13 et "ça marche pas" devient légitime.

##### Étape 2 — `.browserslistrc` (exo 3)

```text
# .browserslistrc — projet Agroalimentaire / Dashboard partenaires
# Cible parc : Chrome internes + BYOD sous-traitants (desktop + mobile)
# Engagement contractuel : navigateurs stables sortis dans les 24 derniers mois
# Plancher = versions sorties fin 2023 (≈ 18-24 mois de recul à date)
# Revue : 2026-05-17 — relecture tous les 12 mois (cible glissante)

Chrome >= 120
Edge >= 120
Firefox >= 121
Safari >= 17
iOS >= 17
not dead
```

Justification :

- Plancher 120 pour Chromium (sorti fin 2023, > 18 mois de recul).
- **Firefox 121** précisément : c'est la première version Firefox avec `:has()` (déc 2023). Plancher en-dessous = `:has()` casse pour les sous-traitants sur Firefox ESR.
- **Safari 17 / iOS 17** : aligne sur la fenêtre Baseline Widely Subgrid + `:has()` natifs.
- **Pas de `last N versions`** ici non plus : on engage le client sur du chiffrable. Le `last` deviendra ambigu en revue 12 mois.

##### Étape 3 — Tableau de décision (exo 3, plancher commun)

| Feature | Première version OK (toutes plateformes) | Statut cible (plancher commun) | Décision |
| --- | --- | --- | --- |
| `@layer` | Chrome 99 / FF 97 / Safari 15.4 | Plancher 120/121/17 ≫ tous seuils | ✅ Utiliser directement |
| `:has()` | Chrome 105 / FF 121 / Safari 15.4 | Plancher 121 = seuil FF (+0) | ✅ Utiliser, **mais plancher Firefox calé pile** — surveiller en revue |
| `@container` | Chrome 105 / FF 110 / Safari 16 | Plancher ≫ tous seuils | ✅ Utiliser directement |
| `oklch()` | Chrome 111 / FF 113 / Safari 15.4 | Plancher ≫ tous seuils | ✅ Utiliser directement |
| Subgrid | Chrome 117 / FF 71 / Safari 16 | Plancher ≫ tous seuils | ✅ Utiliser directement |

##### Étape 4 — Bilan (exo 3)

Si **la clause "navigateurs stables ≤ 24 mois" est validée par mail**, les 5 features passent. C'est **l'engagement écrit qui fait la cible**, pas l'intuition.

```css
/* Target: Chrome/Edge >= 120, FF >= 121, Safari/iOS >= 17, not dead
   Scope: Agroalimentaire / Dashboard partenaires BYOD
   Last reviewed: 2026-05-17 — clause contractuelle "navigateurs < 24 mois" requise
   Revue obligatoire annuelle (cible glissante) */
```

> ⚠️ **Variante "si client refuse la clause"** : il faut alors viser le **pire plausible** (Safari 14, Firefox ESR 102…). À ce moment-là, le tableau passe à `@supports` systématique sur `:has()`, `oklch()`, Subgrid. Le coût d'intégration explose — c'est l'argument à présenter au client pour faire signer la clause.

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
> 🎯 **Mantra du module** : *"On ne dit plus 'je crois que ça passe'. On dit 'Baseline widely / cible client OK'."*
