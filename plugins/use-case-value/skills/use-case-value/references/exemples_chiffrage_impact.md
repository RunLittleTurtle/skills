# Exemples de chiffrage Impact $ — règle d'or adoucie v1.2

Ce document aide le LLM à décider, pour chaque colonne Impact $ (cols 13 à 18), ce qui peut entrer dans la cellule (chiffre dur, formule simple sur chiffres durs, ou inférence obvious sur signal cité) versus ce qui doit rester en Notes comme question à poser au sponsor.

## Principe général v1.2 (règle d'or adoucie)

**Va dans la cellule** :
- Valeur citée verbatim par un sponsor ou participant atelier
- Valeur tirée d'un tableau source (Excel, CSV, rapport)
- Formule simple sur chiffres durs cités appliquant un standard de méthode (heures × 80 $/h)
- **Inférence obvious sur signal cité explicitement** convertie via standard de méthode documenté :
  - Citation FTE explicite → 100 000 $/an par FTE saturé (FTE standard)
  - Heures par semaine citées → × 50 sem × 80 $/h (coût horaire standard)
  - Personnes × heures unitaires citées → conversion directe

**Va en Notes (jamais dans la cellule)** :
- Estimation contextuelle du LLM avec multiplicateur inventé ("% captable", "probabilité de", "valeur moyenne contrat" sans citation)
- Extrapolation de volumes sans signal dans les inputs
- Toute valeur dont la dérivation invente un paramètre de conversion

**Test décisif** : "Si on me demande d'où vient cette valeur, est-ce que je peux pointer à une citation verbatim sponsor + un standard de méthode documenté ? Si oui → cellule. Si je dois inventer un paramètre (un %, une probabilité, une valeur unitaire jamais nommée) → cellule = 0, hypothèse en Notes."

---

## Exemples par colonne

### Col 13 — Impact $ Temps Perdu

**Va dans la cellule** :

| Indice dans l'input | Valeur cellule | Justification |
|---------------------|----------------|---------------|
| "20 à 30 heures par semaine de matching" (Shirley) | `100 000 $` | Formule : 25 h/sem × 50 sem × 80 $/h. Anchor heures cité. |
| "Valérie passe 6 heures par soumission, fait 5 soumissions/sem" (Valérie) | `120 000 $` | Formule : 30 h/sem × 50 × 80. Anchors heures et volume cités. |
| "Eric estime 10 h/sem sur la planif manuelle" (Eric) | `40 000 $` | Formule : 10 × 50 × 80. Anchor heures cité. |
| "L'équipe de 5 personnes y passe environ 2 h/jour chacune" (Nicolas) | `200 000 $` | Formule : 5 × 2 × 250 × 80. Anchors équipe et heures cités. |

**Va en Notes** :

| Indice dans l'input | Cellule | Note |
|---------------------|---------|------|
| "On y passe beaucoup de temps" (sans chiffre) | `0` | `À chiffrer : combien d'heures par semaine en moyenne ? À titre indicatif si 15 h/sem × 50 sem × 80 $/h = 60 000 $.` |
| "C'est énorme comme charge" (sans chiffre) | `0` | `À chiffrer : équivalent FTE actuel sur cette tâche ?` |

### Col 14 — Impact $ Opportunités Manquées (très restrictif)

**Va dans la cellule** :

| Indice dans l'input | Valeur cellule | Justification |
|---------------------|----------------|---------------|
| "On perd environ 10 contrats par an à 50 000 $ chacun" (Maxime) | `500 000 $` | Citation directe avec valeur ET volume cités. |
| "Tableau commercial montre 8 leads perdus en 2025, valeur moyenne 75 000 $" | `600 000 $` | Tableau source explicite. |

**Va en Notes** (cas typique) :

| Indice dans l'input | Cellule | Note |
|---------------------|---------|------|
| "Trois semaines au lieu de 24 heures, ça bloque" (Valérie, sans chiffre contrat) | `0` | `À chiffrer : combien de contrats perdus par an ? À titre indicatif si 5 contrats × 100 000 $ × 30 % capture = +150 000 $.` |
| "Les gros clients nous trouvent trop lents" | `0` | `À chiffrer : volume et valeur des appels d'offres perdus.` |

### Col 15 — Impact $ Coût Erreurs

**Va dans la cellule** :

| Indice dans l'input | Valeur cellule | Justification |
|---------------------|----------------|---------------|
| "180 000 $ de non-conformité par an" (Nicolas) | `180 000 $` | Citation directe valeur annuelle. |
| "2 erreurs majeures/an à 15 000 $ chacune en marge perdue" (Maxime) | `30 000 $` | Citation directe volume × coût. |

**Va en Notes** :

| Indice dans l'input | Cellule | Note |
|---------------------|---------|------|
| "180 000 $ non-conformité" + LLM veut appliquer "40 % captable" | `0` | `À chiffrer : quel % de la non-conformité est captable par vision auto ? À titre indicatif si 40 % captable = +72 000 $.` |
| "On fait des erreurs occasionnellement" | `0` | `À chiffrer : taux d'erreur et coût moyen.` |

**Cas frontière critique** : le 180 000 $ cité par le sponsor est l'**impact actuel total**, pas l'impact captable de l'automatisation. Le multiplicateur "% captable" est une hypothèse LLM, donc cellule = 0 et le 40 % captable va en Notes.

### Col 16 — Impact $ Main d'Œuvre / Saturation (adoucissement v1.2 important ici)

**Va dans la cellule (NOUVEAU v1.2)** :

| Indice dans l'input | Valeur cellule | Justification |
|---------------------|----------------|---------------|
| "Sophie est à temps plein sur les soumissions" (Maxime) | `100 000 $` | **NOUVEAU v1.2** : citation FTE explicite ("temps plein" = 1 FTE saturé) → standard FTE 100 000 $/an. |
| "1 personne dédiée au matching PO" (Shirley) | `100 000 $` | **NOUVEAU v1.2** : citation FTE explicite. |
| "Il faut quelqu'un de full-time là-dessus, sinon ça plante" (Nicolas) | `100 000 $` | **NOUVEAU v1.2** : citation FTE explicite ("quelqu'un de full-time" = 1 FTE). |
| "Engagement Direction : on évite 1 FTE chargé à 120 000 $" | `120 000 $` | Citation directe avec valeur explicite (override du standard 100k). |
| "Valérie a fait 200 h sup payées à 60 $/h en 2025" (paie) | `12 000 $` | Donnée source explicite. |

**Va en Notes** (v1.2, plus restrictif qu'on pourrait penser) :

| Indice dans l'input | Cellule | Note |
|---------------------|---------|------|
| "Valérie est saturée" (sans citation FTE explicite ni heures sup chiffrées) | `0` | `À chiffrer : Valérie représente-t-elle vraiment un FTE saturé sur cette tâche ? Si oui = 100 000 $ Main d'Œuvre.` |
| "On ne peut pas grossir sans embaucher" (sans engagement chiffré) | `0` | `À chiffrer : combien de FTE évités si l'automatisation marche ?` |
| "L'expert RH est sollicité trop souvent" (sans heures sup ni FTE) | `0` | `À chiffrer : heures sup ou FTE équivalent ?` |

**Différence clé v1.0/v1.1 → v1.2** : avant, "Sophie est à temps plein" → 0 (pas de citation $ verbatim). Maintenant, "à temps plein" est un anchor FTE explicite qui se convertit avec le standard de méthode à 100 000 $. La traçabilité est préservée : la citation verbatim + le standard documenté pointent au résultat.

### Col 17 — Impact $ Dépendances Externes

**Va dans la cellule** :

| Indice dans l'input | Valeur cellule | Justification |
|---------------------|----------------|---------------|
| "Pénalités contractuelles Sabre 30 000 $/an documentées" | `30 000 $` | Document source explicite. |
| "Retards fournisseur causent 50 000 $ de stockage supplémentaire/an" | `50 000 $` | Citation valeur explicite. |

**Va en Notes** :

| Indice dans l'input | Cellule | Note |
|---------------------|---------|------|
| "Sabre est lent" (sans pénalité chiffrée) | `0` | `À chiffrer : coûts directs des retards Sabre.` |
| "On dépend de fournisseurs externes" | `0` | `À chiffrer : volume et coût des blocages annuels.` |

### Col 18 — Impact $ Frictions Inter-Départementales

**Va dans la cellule** :

| Indice dans l'input | Valeur cellule | Justification |
|---------------------|----------------|---------------|
| "Estimation et Production échangent 5 emails par dossier, 30 min/dossier × 1500 dossiers × 70 $/h" (Maxime) | `52 500 $` | Citation volume + temps + taux. |
| "Comptabilité refait 20 % du travail de Réception, 10 h/sem chacune" (Shirley) | `40 000 $` | Citation volume et temps. |

**Va en Notes** :

| Indice dans l'input | Cellule | Note |
|---------------------|---------|------|
| "Il y a beaucoup d'allers-retours" (sans chiffre) | `0` | `À chiffrer : combien d'échanges et temps consommé en moyenne ?` |

---

## Frontière critique : chiffre dur vs hypothèse LLM (rappel v1.2)

**Test rapide** : "Pour cette valeur de cellule, est-ce que tous les paramètres de la formule sont :
- (a) cités verbatim dans les inputs OU
- (b) un standard de méthode documenté (80 $/h, 100 000 $/FTE) ?

Si OUI les deux → cellule.
Si NON, il y a un paramètre inventé (% captable, probabilité, valeur unitaire non citée) → cellule = 0, hypothèse en Notes."

Exemples qui transforment une formule en hypothèse (cellule = 0) :
- **% captable** d'un pain actuel
- **Probabilité de capture** d'opportunités
- **Valeur moyenne** contrat sans citation
- **Multiplicateur de gain** sur un coût existant

Exemples qui restent dans la cellule (v1.2 adoucie) :
- **Heures par semaine** citées par sponsor + 80 $/h standard
- **FTE saturé** cité par sponsor + 100 000 $ standard
- **Valeur annuelle directe** citée par sponsor
- **Volume × valeur unitaire** tous deux cités par sponsor

## Standards de méthode (résumé)

**Coût horaire fully-loaded** : 80 $/h. Appliqué seulement si heures citées explicitement.

**FTE chargé fully-loaded** : 100 000 $/an. Appliqué seulement si saturation FTE citée explicitement ("temps plein", "1 FTE", "dédié", "full-time"). Cohérent avec 80 $/h × 1250 h productives/an.

Si le sponsor cite un taux ou profil spécifique différent, utiliser cette valeur et documenter la source dans Notes.

## Interaction avec Step 2bis Root Cause et col 27 Impact Potentiel

Step 2bis identifie les root causes actionnables, les symptômes se consolident en Pain Points sur la ligne de la root cause. L'Impact $ chiffré capture l'impact agrégé sans double-compter.

Toutes les estimations indicatives chiffrées présentes dans Notes sous "À chiffrer en atelier" alimentent la col 27 Impact Potentiel Estimé qui entre dans le Score avec discount 0.3. Donc les use cases haut potentiel non encore chiffré ne tombent plus à Score 0 dans le tri.
