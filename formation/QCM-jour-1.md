# 📝 QCM — Fin Jour 1 (10 min)

> Auto-évaluation. Une seule bonne réponse sauf mention contraire. Corrigé en bas.

---

## 1. Cascade CSS — ordre des étapes

Dans quel ordre le moteur CSS arbitre-t-il deux règles en conflit ?

- A. Spécificité → ordre → couche → origine
- B. Origine + importance → couche → spécificité → ordre d'apparition
- C. Spécificité → !important → couche → ordre
- D. Couche → spécificité → origine → ordre

## 2. `@layer` — règle d'or

Soient deux couches déclarées `@layer framework, overrides;`. Une règle `color: red` (spécificité 0,1,0) en couche `overrides` et `color: blue` (spécificité 0,5,0) en couche `framework`. Couleur finale ?

- A. blue (spécificité plus forte gagne)
- B. red (couche déclarée plus tard gagne)
- C. dépend de l'ordre d'apparition dans le fichier
- D. erreur — la spécificité 0,5,0 court-circuite les couches

## 3. Couche implicite

Une règle CSS écrite **hors** de tout `@layer` se place :

- A. dans la première couche déclarée
- B. dans la dernière couche déclarée
- C. dans une couche implicite **après** toutes les couches nommées
- D. avant la couche `reset`

## 4. `:where()`

`:where(.a, .b, .c) p` a une spécificité de :

- A. 0, 3, 1
- B. 0, 1, 1
- C. 0, 0, 1
- D. 0, 1, 0

## 5. Pyramide de couches (formation Dymasco)

L'ordre **correct** des 4 couches enseignées est :

- A. `framework, reset, priority, overrides`
- B. `reset, framework, overrides, priority`
- C. `reset, priority, framework, overrides`
- D. `priority, reset, framework, overrides`

## 6. Importer un framework legacy dans une couche

Quelle syntaxe permet de "ranger" un CSS framework dans la couche `framework` au moment de l'import ?

- A. `@import url("apriso-base.css") @layer(framework);`
- B. `@import url("apriso-base.css") layer(framework);`
- C. `@layer framework { @import url("apriso-base.css"); }`
- D. `@import "apriso-base.css" into framework;`

## 7. `clamp(min, ideal, max)`

`font-size: clamp(14px, 1.2vw + 0.5rem, 22px)` veut dire :

- A. font-size égal toujours à 1.2vw + 0.5rem
- B. font-size compris entre 14px et 22px, valeur idéale fluide entre les deux
- C. min(14px, max(1.2vw, 22px))
- D. font-size fixe à 22px

## 8. `min(1440px, 100%)` sur un conteneur

Quelle est la largeur réelle ?

- A. jamais plus large que 1440px **et** jamais plus large que le viewport
- B. jamais plus large que 1440px **et** jamais plus large que le **conteneur parent**
- C. 1440px si le parent est plus large, sinon 100% du viewport
- D. la plus grande des deux valeurs

## 9. Flexbox — empêcher un item de rétrécir

Quelle déclaration interdit à un flex item de descendre sous sa taille de contenu ?

- A. `flex-shrink: 1`
- B. `min-width: 0`
- C. `flex-shrink: 0` (ou `min-width: auto` selon le cas)
- D. `flex-basis: 0`

## 10. Grid — `display: contents` sur `<table>`

Cocher la **bonne** raison **valide** d'utiliser `display: contents` sur un `<table>` :

- A. tableau de données exporté en CSV par les utilisateurs
- B. KPI Board purement présentationnel, sans sémantique tabulaire
- C. tableau trié dynamiquement par colonne
- D. tableau lu en navigation cellule par cellule par un lecteur d'écran

---

## ✅ Corrigé

1. **B** — origine + importance, couche, spécificité, ordre
2. **B** — couche déclarée en dernier gagne, même avec spécificité plus faible
3. **C** — règles hors `@layer` = couche implicite **finale**, donc plus forte
4. **C** — `:where()` met la spécificité de son contenu à 0, reste `p` = 0,0,1
5. **C** — `reset, priority, framework, overrides`
6. **B** — syntaxe `@import url(...) layer(nom);`
7. **B** — clamp = min, idéal, max
8. **B** — `100%` réfère au **conteneur parent**, pas au viewport
9. **C** — `flex-shrink: 0` (et attention au `min-width: auto` qui peut autoriser le rétrécissement contenu)
10. **B** — réservé aux tableaux présentationnels, jamais sur données tabulaires sémantiques
