# 🎓 Formation CSS avancé — Dymasco — 18-19 mai 2026

## 🎯 Objectifs de formation

À l'issue de cette formation, vous serez capable d'intervenir **en contexte industriel** sur une application existante (Dymasco / DELMIA Apriso) en appliquant une **méthodologie d'overrides CSS** : corriger, étendre et adapter l'UI **sans polluer** le code d'origine ni déclencher des régressions en cascade.

- Mettre en place une **stratégie d'overrides maintenable** : isolation des surcharges, organisation des couches, règles de nommage, variables et points d'extension.
- **Arbitrer** l'usage des fonctionnalités CSS modernes (Grid/Subgrid, Container Queries, propriétés logiques, etc.) **selon le parc navigateur réel** (Windows LTSC, tablettes d'atelier, contraintes IT) en s'appuyant sur des outils factuels : **Baseline**, **Can I Use**, **Browserslist** (et une politique de dégradation/contournement).
- Diagnostiquer et résoudre les **guerres de cascade** : spécificité incontrôlée, héritage inattendu, styles inline, IDs dynamiques, frameworks/legacy, sans tomber dans une escalade systématique de `!important`.
- **Briser proprement les verrous de spécificité** (scopage, couches, sélecteurs ciblés, stratégie `!important` maîtrisée si nécessaire) en gardant un CSS lisible, traçable et réversible.

## 👥 Public & Prérequis

- Développeurs / intégrateurs / concepteurs web seniors.
- Prérequis : maîtriser les bases HTML/CSS, avoir une pratique régulière d'intégration.

## 🛠️ Modalités pédagogiques

- Méthode active : alternance théorie courte → démonstration → exercices.
- **Fil rouge unique** : un dashboard de supervision (Forge × Pâtes Mamie Lulu) évolue module par module, chaque feature CSS résout un problème concret du fichier `apriso-base.css`.
- Chaque chapitre inclut : pièges courants, compatibilité navigateurs, ressources.
- Checkpoint `overrides.css` fourni après chaque module pour valider l'état attendu.

## 📚 Ressources transverses

- [MDN Web Docs (FR)](https://developer.mozilla.org/fr/docs/Web/CSS)
- [web.dev/learn/css](https://web.dev/learn/css)
- [caniuse.com](https://caniuse.com)
- [Baseline](https://web.dev/baseline)
- [State of CSS](https://stateofcss.com)
- [CSS-Tricks](https://css-tricks.com)

## 🗺️ Plan de formation (2 jours — 14h)

### Jour 1 — matin (3h45)

- [🧭 00 — Cible navigateur & arbitrage](./00-cible-navigateur.md) — 1h00
- [🏗️ 01 — Architecture CSS pour overrides](./01-architecture-css.md) — 2h00
- [🔤 02 — Typographie & dimensions fluides](./02-typographie-fluide.md) — 0h45

### Jour 1 — après-midi (3h30)

- [🤸 03 — Flexbox avancé](./03-flexbox-avance.md) — 1h15
- [🧱 04 — Grid Layout, Subgrid & layouts denses](./04-grid-subgrid.md) — 2h15
- [📝 QCM — fin Jour 1](./QCM-jour-1.md) — 10 min (auto-évaluation)

### Jour 2 — matin (3h15)

- [🧠 05 — Sélecteurs modernes & propriétés logiques](./05-selecteurs-modernes.md) — 1h45
- [📦 06 — Container Queries](./06-container-queries.md) — 0h45
- [🎨 07 — Couleurs modernes & thèmes](./07-couleurs-themes.md) — 0h45

### Jour 2 — après-midi (3h15)

- [📱 08 — Media Queries fonctionnelles](./08-media-queries-fonctionnelles.md) — 1h15
- [🎞️ 09 — SVG & micro-animations d'état](./09-svg-animations-etat.md) — 1h00
- [⚡️ 10 — Performance & Accessibilité](./10-perf-a11y.md) — 1h00
- [📝 QCM — fin Jour 2](./QCM-jour-2.md) — 10 min (auto-évaluation)
