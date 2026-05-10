# Projet fil rouge — Forge × Pâtes Mamie Lulu

> 🏭 Le projet que vous allez construire pendant les 2 jours de formation.
> Toutes les notions vues en cours s'appliquent ici, sur un cas concret type.

---

## 🎬 Le contexte

### Vous êtes chez Dymasco

Votre équipe lance **Forge**, un nouveau module de la gamme DymaPacks™ : un dashboard temps réel de supervision de ligne de production prêt à déployer dans DELMIA Apriso. Forge embarque une bibliothèque de composants front pré-stylés (header, KPI, cards machines, journal d'événements) que les intégrateurs personnalisent ensuite à l'identité visuelle de chaque client.

### Le client : Pâtes Mamie Lulu

ETI familiale de pâtes artisanales, 3 lignes de production, 8 machines critiques, 120 collaborateurs. Modernisation du SI industriel en cours, premier déploiement Apriso. Lulu (la directrice de prod, petite-fille de la fondatrice) a deux exigences :

- 👁️ Un dashboard **lisible depuis l'atelier** : forte luminosité, opérateurs qui portent des gants.
- 🍝 Une identité visuelle **chaleureuse** qui fait honneur à l'esprit familial — pas un dashboard "froid" d'usine pétrochimique.

### Votre mission

Vous êtes l'intégrateur côté Dymasco. Vous recevez `apriso-base.css` du framework Forge et vous **ne pouvez pas le modifier** : il est partagé avec tous les autres clients. Vous écrivez `overrides.css` pour transformer le dashboard générique en dashboard "Mamie Lulu", en CSS natif moderne, sans toucher au HTML.

---

## 📁 Structure du projet

```
projet-fil-rouge/
├── README.md              ← vous êtes ici
├── 00-base/               ← état initial (point de départ)
│   ├── index.html         ← structure HTML imposée (NE PAS MODIFIER)
│   ├── apriso-base.css    ← CSS du framework Forge (NE PAS MODIFIER)
│   └── overrides.css      ← votre fichier de travail (vide au départ)
└── checkpoints/           ← état attendu après chaque module
    ├── 01-architecture/
    ├── 02-typography/
    └── ...
```

À chaque module, vous enrichissez `overrides.css`. À la fin du J2, vous aurez un dashboard complet et déployable.

---

## 🎯 Règles du jeu

| ✅ Vous pouvez | ❌ Vous ne pouvez pas |
|---|---|
| Écrire du CSS dans `overrides.css` | Modifier `index.html` |
| Créer des fichiers CSS additionnels | Modifier `apriso-base.css` |
| Déclarer des variables CSS, `@layer`, `@media` | Utiliser un préprocesseur (Sass, Less) |
| Utiliser tout CSS supporté par Edge/Chromium ≤3 ans | Utiliser du JavaScript |

> ⚠️ Pourquoi pas de JS ? Parce que c'est une formation **CSS**. Tout ce qu'on va faire passe en CSS pur. Si vous trouvez un cas où il vous faudrait du JS, levez la main : c'est l'occasion d'apprendre une technique CSS moderne que vous ne connaissiez peut-être pas.

---

## 🧰 Environnement de travail recommandé

- VS Code (ou équivalent) avec l'extension Live Server
- Edge ou Chrome récent (DevTools requis)
- Un dossier de travail isolé (vous allez itérer 11 fois)

```bash
# Cloner ou copier le dossier 00-base/ dans votre espace de travail
cp -r 00-base/ mon-travail/
cd mon-travail/
# Ouvrir index.html avec Live Server
```

---

## 🗺️ Plan des 2 jours

| Jour | Modules | Ce que vous ajoutez au dashboard |
|---|---|---|
| **J1 matin** | 0 — Cible navigateur | Décider de la cible Forge |
| | 1 — Architecture CSS | Tokens, `@layer`, nesting, premières surcharges propres |
| | 2 — Typo & dimensions fluides | KPI lisibles, hiérarchie typo |
| **J1 aprem** | 3 — Flexbox avancé | Header propre, gestion textes débordants |
| | 4 — Grid Layout & layouts denses | Bandeau KPI, grille machines, sticky, tableaux |
| **J2 matin** | 5 — Sélecteurs modernes | Mises en évidence d'alertes (`:has()`) |
| | 6 — Container Queries | Card-Machine adaptative |
| | 7 — Couleurs & thèmes | Identité Mamie Lulu finalisée |
| **J2 aprem** | 8 — Media Queries fonctionnelles | Mode nuit, tactile gants, impression OF |
| | 9 — SVG & animations d'état | Pictos états, transitions sobres |
| | 10 — Perf & A11y | Focus visible, contraste, mise en prod |

---

## 💡 Conseil pour bien suivre

Chaque module a sa **section "exercice fil rouge"** qui décrit précisément ce que vous devez ajouter à `overrides.css`. Si vous décrochez sur un module, vous pouvez reprendre proprement au module suivant en repartant du checkpoint correspondant.

> 🎓 Bon stage, et que la cascade soit avec vous.
