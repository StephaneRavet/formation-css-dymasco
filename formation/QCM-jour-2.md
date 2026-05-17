# 📝 QCM — Fin Jour 2 (10 min)

> Auto-évaluation. Une seule bonne réponse sauf mention contraire. Corrigé en bas.

---

## 1. `:has()` — spécificité

Spécificité de `.card:has(#header .badge)` ?

- A. 0, 2, 0
- B. 1, 2, 0
- C. 1, 1, 0
- D. 0, 1, 0 (`:has()` annule la spé interne)

## 2. `:contains()` en CSS

Le sélecteur `:contains("texte")` :

- A. existe en CSS standard, widely available
- B. existe uniquement dans le draft Selectors 5
- C. **n'existe pas** en CSS — uniquement jQuery/Sizzle
- D. existe mais nécessite `@supports`

## 3. Propriétés logiques — équivalent RTL

`margin-left: 12px` devient en propriété logique :

- A. `margin-inline-start: 12px`
- B. `margin-block-start: 12px`
- C. `margin-inline-end: 12px`
- D. `margin-start: 12px`

## 4. Container queries — pré-requis

Pour qu'un composant puisse être interrogé par `@container`, son **parent** doit avoir :

- A. `container-type: inline-size` (ou `size`)
- B. `display: grid`
- C. `position: relative`
- D. `contain: layout`

## 5. `oklch()` vs `hsl()`

Pourquoi préférer `oklch()` à `hsl()` pour un design system ?

- A. `oklch()` supporte les couleurs P3 / wide gamut
- B. `oklch()` est perceptuellement uniforme (variations de luminosité prévisibles)
- C. `oklch()` est mieux supporté par Safari
- D. **A et B**

## 6. `color-mix()` — usage typique

`color-mix(in oklab, var(--brand) 80%, white)` produit :

- A. la couleur brand assombrie de 20 %
- B. la couleur brand mélangée à 20 % de blanc (donc éclaircie)
- C. une erreur — `color-mix` n'accepte pas `var()`
- D. la couleur blanche teintée à 20 % de brand

## 7. `@media (hover: hover) and (pointer: fine)`

Pourquoi combiner les deux conditions ?

- A. par habitude historique
- B. parce qu'une tablette hybride (Surface) match `(hover: hover)` seul → ciblage trop large
- C. `(pointer: fine)` est obsolète mais requis pour Safari
- D. `(hover: hover)` ne fonctionne pas sans `(pointer: fine)`

## 8. `prefers-reduced-motion` — implémentation

Bloc correct pour respecter la préférence utilisateur ?

- A. désactiver toutes les animations sans condition
- B. `@media (prefers-reduced-motion: reduce) { *, *::before, *::after { animation: none !important; transition: none !important; } }`
- C. utiliser un JS qui détecte le système
- D. ne rien faire — c'est au navigateur

## 9. `content-visibility: auto` — piège

Sur quel type de contenu est-ce **risqué** ?

- A. cards d'images chargées en lazy
- B. longue liste statique de logs (1000+ lignes)
- C. texte critique recherché par **Ctrl+F** ou lu par lecteur d'écran
- D. footer

## 10. `print-color-adjust: exact`

Bonne pratique pour préserver les couleurs à l'impression ?

- A. `* { print-color-adjust: exact }` partout dans `@media print`
- B. cibler uniquement les éléments sémantiquement colorés (badges, statuts)
- C. inutile — les navigateurs gèrent
- D. uniquement supporté par Chromium

---

## ✅ Corrigé

1. **B** — `:has()` prend la spécificité du plus spécifique dans sa liste : `#header` = 1,0,0, plus `.card` = 1,1,0, plus `.badge` = 1,2,0
2. **C** — `:contains()` n'existe **pas** en CSS, jamais (jQuery uniquement)
3. **A** — `margin-inline-start` (suit la direction du texte)
4. **A** — `container-type: inline-size` sur le parent (ou `size`)
5. **D** — wide gamut **et** perceptuellement uniforme
6. **B** — `var(--brand)` à 80 %, blanc à 20 % → brand éclaircie
7. **B** — tablettes hybrides Surface matchent `(hover: hover)` seul, ciblage trop large sans `(pointer: fine)`
8. **B** — bloc reset universel avec `!important` (le seul `!important` toléré)
9. **C** — texte recherché Ctrl+F ou lu par lecteur d'écran peut être skippé
10. **B** — cibler badges/statuts uniquement, jamais en wildcard `*` (surconsommation d'encre)
