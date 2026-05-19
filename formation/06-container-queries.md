---
formateur:
  prerequis_formateur:
    - savoir différence container-type inline-size vs size (size casse souvent layout)
    - connaître unités cq* (cqi = 1% inline-size conteneur)
    - savoir qu'une @container sans container-type quelque part = SILENCE total
    - widely available 2023 — OK Dymasco
  ressources_demo_a_preparer:
    - checkpoint Module 5 ouvert
    - DevTools prêt pour resize la sidebar (modifier width inline)
    - tableau récap container-type / cq* affiché
  pitch_ouverture: >
    "Le Saint Graal du composant réutilisable. La même Card-Machine dans la sidebar
    (200px), la grille (280px), l'écran mural (500px) — un seul CSS. Avant : 3 variantes
    BEM. Maintenant : le composant interroge SA place."
  energie_attendue: haute — module court, effet "wow" immédiat, plaisir formateur
  duree_cible: 45 min — découpage 15 concepts / 10 démo / 15 exo / 5 récap
  variantes_timing:
    si_en_retard: zapper bonus style queries + imbrication conteneurs
    si_en_avance: faire trouver 3 composants candidats dans LEUR code actuel
  points_a_marteler:
    - "@container interroge LE CONTENEUR, pas le viewport"
    - "container-type: inline-size 95% du temps — size casse layout"
    - "container-name optionnel mais RECOMMANDÉ (lisibilité + scoping)"
    - "cqi = 1% inline-size conteneur (vs vw = viewport)"
    - "tester en resizant le CONTENEUR, pas le viewport — sinon démo ratée"
  pieges_apprenants:
    - oublier container-type sur l'ancêtre → @container silencieux jamais déclenché
    - mettre container-type: size "par sécurité" → conteneur perd hauteur intrinsèque
    - penser cqw = viewport width (c'est conteneur)
    - interroger un parent non déclaré → remonte au plus proche déclaré
  questions_probables:
    - q: "Quand utiliser Container vs Media Queries ?"
      r: composant réutilisable dans plusieurs zones = container. Layout global = media.
    - q: "Imbriquer plusieurs conteneurs ça marche ?"
      r: oui — grille hôte → card hôte → metric hôte. Use container-name pour ciblage.
    - q: "Style queries c'est utilisable ?"
      r: Newly 2024 — éviter Dymasco si cible client variable.
    - q: "Impact perf ?"
      r: négligeable jusqu'à plusieurs centaines de conteneurs. Mesurer si doute.
  transition_module_suivant: >
    "Composants adaptatifs : OK. Maintenant identité visuelle. Hex statique → oklch
    perceptuel + color-mix() dynamique. Module 7 : une teinte source, tout suit."
---

# Module 6 — Container Queries

> ⏱️ **Durée** : 0h45 — **J2 matin**
> 🧭 **Type** : module court, fort effet "wow"
> 🎯 **Checkpoint** : `projet-fil-rouge/checkpoints/06-container-queries/overrides.css`

---

## 🎯 Objectif (en 1 phrase)

Faire en sorte qu'un **même composant** (la Card-Machine) s'adapte automatiquement à la **taille de son conteneur**, pas à la taille du viewport.

---

## ⚙️ Pour qui c'est utile chez Dymasco

Vous concevez des **DymaPacks réutilisables** : la même Card-Machine doit pouvoir être utilisée :

- Dans la **grille principale** (large, ~280px) → version riche : header + 2 metrics + dernier événement.
- Dans la **sidebar** (étroite, ~200px) → version compacte : header + 1 metric.
- Sur l'**écran mural** (très large, ~500px) → version XL : header + 3 metrics + graphique.

Avant Container Queries → 3 fichiers CSS, 3 variantes BEM (`--small`, `--medium`, `--large`), entretien lourd.

Après → **un seul composant**, qui se réorganise selon la place qu'on lui donne.

C'est le **Saint Graal du composant réutilisable**.

---

## 📚 Concepts (15 min)

### 1. Pourquoi pas Media Queries ? (3 min)

Media queries répondent à **la taille du viewport**, pas à celle du conteneur.

Si votre Card-Machine est dans une sidebar de 200px alors que le viewport fait 1920px, une `@media (min-width: 800px)` est **vraie** alors que la card est minuscule. Le composant ne sait pas où il est.

Container Queries inversent la perspective : le composant **interroge son conteneur**.

### 2. Déclarer un conteneur (4 min)

```css
.apriso-machine-grid {
  container-type: inline-size;     /* on s'intéresse à la largeur */
  container-name: machine-zone;    /* optionnel mais recommandé */
}
```

| `container-type` | Sens |
|---|---|
| `normal` | Pas de container query (défaut) |
| `inline-size` | Tracker la largeur (axe inline) — **le plus utilisé** |
| `size` | Tracker largeur ET hauteur (impose un layout 2D, plus rare) |

> ⚠️ `container-type: size` impose au conteneur une **taille intrinsèque connue** → souvent casse le layout (le conteneur ne grandit plus avec son contenu). Utiliser `inline-size` 95% du temps.

### 3. Interroger le conteneur (5 min)

```css
@container machine-zone (min-width: 320px) {
  .apriso-machine-card__metrics {
    grid-template-columns: 1fr 1fr;   /* 2 colonnes si conteneur >= 320px */
  }
}

@container machine-zone (min-width: 480px) {
  .apriso-machine-card__metrics {
    grid-template-columns: 1fr 1fr 1fr;   /* 3 colonnes si très large */
  }
  .apriso-machine-card__chart { display: block; }
}
```

Sans `container-name`, on interroge le conteneur **le plus proche** :

```css
@container (min-width: 320px) { ... }
```

#### Cible Baseline

Container Queries (`container-type`, `@container`) : **Widely available** depuis 2023 (Chrome/Edge 105+, Safari 16+, Firefox 110+). Pour Dymasco : ✅.

### 4. Unités relatives au conteneur (3 min)

Comme `vw`/`vh` pour le viewport, il existe `cqw`/`cqh`/`cqi`/`cqb` pour le conteneur.

| Unité | Sens |
|---|---|
| `cqw` | 1% de la largeur du conteneur |
| `cqh` | 1% de la hauteur |
| `cqi` | 1% de l'inline-size (= largeur en writing-mode horizontal) |
| `cqb` | 1% du block-size (hauteur en horizontal) |
| `cqmin` / `cqmax` | min/max des deux dimensions |

```css
.apriso-machine-card__title {
  font-size: clamp(13px, 4cqi, 18px);   /* 4% de la largeur du conteneur, borné */
}
```

→ Le titre grossit avec la card, **pas avec l'écran**. Exactement ce qu'on veut.

---

## 🔍 Démonstration (10 min)

**Live** : faire varier la largeur d'une zone hôte de Card-Machine, voir la card se reconfigurer.

### Étape 1 — Setup (3 min)

Déclarer la grille comme conteneur :

```css
@layer overrides {
  .apriso-machine-grid {
    container-type: inline-size;
    container-name: machine-zone;
  }
}
```

### Étape 2 — Card adaptative (5 min)

Trois paliers :

```css
@layer overrides {
  /* Compact par défaut */
  .apriso-machine-card__metrics {
    display: grid;
    grid-template-columns: 1fr;     /* 1 colonne par défaut */
  }

  /* Standard */
  @container machine-zone (min-width: 320px) {
    .apriso-machine-card__metrics {
      grid-template-columns: 1fr 1fr;
    }
  }

  /* XL */
  @container machine-zone (min-width: 480px) {
    .apriso-machine-card__metrics {
      grid-template-columns: 1fr 1fr 1fr;
    }
  }
}
```

### Étape 3 — Effet wow (2 min)

DevTools : passer le `<aside>` (sidebar) à `width: 600px` → cards intérieures basculent en mode XL. Repasser à `width: 200px` → mode compact. **Une seule règle CSS** pilote tout.

---

## 🛠️ Exercice fil rouge (15 min)

### Consigne

Reprendre le checkpoint Module 5 et :

1. Déclarer `.apriso-machine-grid` comme conteneur (`container-type: inline-size`, nom `machine-zone`).

2. Sur `.apriso-machine-card__metrics` (le `dl` qui contient les 2 metrics OEE / Disponibilité dans la card) :
   - Par défaut : 1 colonne, gap réduit.
   - `@container (min-width: 280px)` : 2 colonnes côte à côte.

3. Sur `.apriso-machine-card__last-event` :
   - Par défaut : caché (`display: none`) — gain de place dans card étroite.
   - `@container (min-width: 280px)` : `display: block`, line-clamp 2 (Module 3).

4. Sur `.apriso-machine-card__title` :
   - Font-size en `clamp(13px, 4cqi, 18px)` → suit la largeur du conteneur, pas de l'écran.

5. **Test à faire** : DevTools, faire varier la **largeur de la sidebar** ou de la **grille machines** en direct. Observer que les cards **internes** se reconfigurent. Faire varier le viewport seul → les cards ne bougent **pas** (la grille reste à la même taille).

### Pièges fréquents

- ❌ Oublier `container-type` → la `@container` query ne se déclenche jamais (silencieux).
- ❌ Mettre `container-type: size` "par sécurité" → conteneur qui n'a plus de hauteur intrinsèque, layout cassé.
- ❌ Confondre `container-name` (déclaration) et nom dans la query (`@container <name>`).
- ❌ Penser que `cqw` réfère au viewport → c'est **le conteneur**.
- ❌ Vouloir interroger un parent **non déclaré** → la query remonte au plus proche déclaré, ou échoue.

### Corrigé attendu

Voir `projet-fil-rouge/checkpoints/06-container-queries/overrides.css`.

---

## 🚀 Pour aller plus loin

### Bonus 1 — Style queries (`@container style(...)`)

Très récent (Newly available 2024). Permet de styler selon une **valeur de variable CSS** sur le conteneur :

```css
@container style(--theme: dark) {
  .apriso-machine-card { color: white; }
}
```

Pour Dymasco aujourd'hui : à éviter (cible client variable).

### Bonus 2 — Imbriquer plusieurs conteneurs

Un composant peut être **conteneur** ET **enfant** d'un autre conteneur. Permet de hiérarchiser : "grille hôte → card hôte → metric hôte".

### Bonus 3 — Combiner avec `clamp()`

```css
.apriso-machine-card__value {
  font-size: clamp(14px, 6cqi, 32px);
}
```

→ Borne la valeur "en cqi" → meilleur des deux mondes (fluide + sécurisé).

### Ressources

- [MDN — Container Queries](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_containment/Container_queries)
- [MDN — container-type](https://developer.mozilla.org/en-US/docs/Web/CSS/container-type)
- [MDN — @container](https://developer.mozilla.org/en-US/docs/Web/CSS/@container)
- Una Kravets, [Container Queries are coming](https://web.dev/blog/cq-stable)

---

## 📝 Récap & checklist

- [ ] Je sais expliquer la différence entre `@media` (viewport) et `@container` (conteneur)
- [ ] Je déclare `container-type: inline-size` (et **pas** `size`) par défaut
- [ ] Je nomme mes conteneurs avec `container-name`
- [ ] Je connais l'unité `cqi` (= 1% de la largeur du conteneur)
- [ ] Je teste mes composants en faisant varier la largeur du conteneur, **pas** du viewport

> 🎯 **Mantra du module** : *"Mon composant ne demande plus 'quelle est la taille de l'écran ?' mais 'combien de place tu me donnes ?'"*
