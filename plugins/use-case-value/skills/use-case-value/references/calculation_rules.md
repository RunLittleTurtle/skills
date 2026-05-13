# Règles de calcul use-case-value v1.0

Ce document détaille les formules, défauts et grille de verdict du CSV 25 colonnes.

## Règle d'or absolue sur les estimations

**Cellules Impact $ (colonnes 13 à 18) acceptent uniquement** :
1. Une valeur citée explicitement par un sponsor, un participant atelier ou une table source.
2. Une valeur dérivée par formule simple sur des chiffres durs (ex : heures par semaine citées × 50 sem × coût horaire fully-loaded par défaut).
3. Sinon : **0**.

**Une cellule à 0 ne signifie pas "pas d'impact".** Elle signifie "non chiffré dans les inputs". Toute cellule à 0 doit avoir une question correspondante structurée dans Notes (col 25) sous "À chiffrer en atelier".

**Toute estimation LLM va exclusivement dans Notes**, jamais dans une cellule Impact $. Le LLM peut proposer une valeur indicative ("à titre indicatif, si X alors environ Y") mais cette valeur reste dans Notes en tant que question à poser au sponsor.

## Coût horaire fully-loaded par défaut

`80 $/h` — profil mixte employé québécois, charges sociales et overhead inclus.

Utilisable pour convertir des heures dures en Impact $ Temps Perdu :
```
Impact $ Temps Perdu = heures par semaine citées × 50 semaines × 80 $/h
```

Modifiable selon le client si une autre référence est fournie en input (ex : taux horaire spécifique à un département mentionné par le sponsor, ou contexte non québécois). Documenter le taux utilisé dans Notes.

## Formules pour les colonnes calculées

### Col 19 — Impact Total Documenté ($/an)

```
Impact Total Documenté = SUM(col13 ; col14 ; col15 ; col16 ; col17 ; col18)
```

Reflète uniquement les chiffres durs cités ou les formules simples sur chiffres durs. Une valeur faible avec questions chiffrables en Notes est plus honnête qu'une valeur gonflée par estimations LLM.

### Col 23 — Score Priorité Impact

```
Score Priorité Impact (brut) =
   Impact Total Documenté
 × Complétude Chiffrage (extraite du label, valeur 1, 2 ou 3)
 × Urgence Stratégique (extraite du label, valeur 1, 2 ou 3)
 × (1 + 0.2 × log(MAX(1 ; Personnes Affectées)))

Score Priorité Impact (normalisé) =
   Score brut / MAX(Score brut sur toutes les lignes du CSV)
```

Le résultat normalisé est compris entre 0 (use case sans impact documenté) et 1 (use case avec le plus fort score documenté du CSV).

**Logique de pondération** :
- Impact Total Documenté est le levier principal du score.
- Complétude Chiffrage récompense les use cases bien documentés (évite que le LLM gonfle des hypothèses pour faire monter le score).
- Urgence Stratégique applique le Cost of Delay (WSJF).
- Personnes Affectées en logarithme : un use case qui touche 50 personnes pèse plus qu'un qui touche 5, avec un effet décroissant. log(50) ≈ 1.70, log(5) ≈ 0.70, log(1) = 0. Le facteur reste raisonnable même pour une organisation complète.

### Col 24 — Verdict Impact

Lookup sur la grille suivante (premier match l'emporte, ordre haut vers bas) :

| Verdict | Conditions (toutes doivent être vraies) |
|---------|------------------------------------------|
| **Critique** | Impact Total Documenté supérieur à 200 000 $ ET Urgence Stratégique ≥ 2 ET Complétude Chiffrage ≥ 2 |
| **Forte** | Impact Total Documenté supérieur à 100 000 $ ET Complétude Chiffrage ≥ 2 |
| **À chiffrer prioritairement** | Impact Total Documenté inférieur à 100 000 $ ET Notes mentionne un Impact potentiel supérieur à 100 000 $ ET Urgence Stratégique ≥ 2 ET sponsor actif détecté |
| **À chiffrer secondairement** | Impact Total Documenté inférieur à 100 000 $ ET Notes mentionne un Impact potentiel inférieur à 100 000 $ |
| **Stratégique long-terme** | Urgence Stratégique = 3 ET Impact Total Documenté faible (sous 100 000 $) ET pas d'estimation forte en Notes |
| **Faible impact** | Impact Total Documenté inférieur à 50 000 $ ET Urgence Stratégique inférieure à 3 ET pas d'Impact potentiel élevé en Notes |

**Détection "sponsor actif"** : le sponsor est cité avec un verbe d'action en atelier ("je pousse", "j'ai besoin", "c'est moi qui en parle", "cela me bloque"), ou un OKR explicite, ou une date butoir mentionnée.

**Détection "Impact potentiel en Notes"** : présence de questions structurées "À chiffrer" dans Notes avec valeur indicative chiffrée :
- Seuil "potentiel supérieur à 100 000 $" : au moins une estimation indicative > 100 000 $
- Seuil "potentiel inférieur à 100 000 $" : estimations présentes mais < 100 000 $

Les seuils 200 000 $, 100 000 $ et 50 000 $ sont calibrés sur l'expérience MIA Innovation auprès de clients PME québécois. À ajuster pour grandes entreprises ou autres contextes.

## Lignes FORMULES et SOURCE (rows 2 et 3 du CSV)

Le gabarit `csv_template.csv` contient :

- **Row 1** : header avec les 25 noms de colonnes.
- **Row 2 (FORMULES)** : pour chaque colonne calculée, la formule de calcul écrite en notation Coda ou Google Sheets pour copier-coller dans un tableur. Pour les colonnes data, "input direct" ou "manuel selon rubrique".
- **Row 3 (SOURCE)** : référence méthodologique (BABOK §10.10, WSJF, DAMA-DMBOK §3, MIA pratique terrain, input atelier).
- **Row 4 et suivantes** : data, une ligne par use case.

Ne jamais modifier les rows 2 et 3 lors du remplissage des use cases. Ce sont la documentation traçable du calcul.

## Col 26 — Dépend de (texte libre, n'entre PAS dans le calcul)

La colonne 26 "Dépend de" capture les use cases prérequis identifiés en Step 2bis du workflow. Elle **n'entre pas dans la formule** du Score Priorité Impact ni dans la grille du Verdict Impact. C'est un signal qualitatif pour la synthèse (section "Dépendances et root causes") et pour aider l'analyste à planifier l'ordre de lancement.

Format texte libre, nom du use case prérequis (matchant exactement la col 1 d'une autre ligne du CSV pour permettre un lookup mécanique), virgules pour séparer plusieurs prérequis.

Exemples :
- Vide (cas par défaut, aucune dépendance inter-use-cases)
- `Data Lake` (une seule dépendance)
- `Data Lake, Workestimity opérationnel` (plusieurs dépendances)

Les dépendances **non-use-case** (intégration tierce externe, condition contractuelle, autre projet hors scope IA) vont en Notes (col 25), pas dans col 26.

## Vérifications de cohérence avant `present_files`

Après remplissage, vérifier ligne par ligne :

- Si col 19 (Impact Total Documenté) > 0 : au moins une cellule des col 13 à 18 doit être > 0 ET citée dans Notes avec source verbatim.
- Si une cellule des col 13 à 18 est à 0 : Notes doit contenir une question "À chiffrer" correspondante avec estimation indicative.
- Si Verdict = "Critique" : col 19 > 200 000 $ ET les cellules > 0 sont toutes citées dans Notes.
- Si Verdict = "À chiffrer prioritairement" : Notes mention au moins une estimation indicative > 100 000 $ ET la source de l'urgence est documentée.
- Si col 22 (Urgence) = 3 : Notes doit citer la source de l'urgence (OKR client verbatim, sponsor avec verbe d'action, date butoir).
- Aucune cellule Impact $ ne doit contenir une valeur "estimée par le LLM sans citation source". Si le LLM a estimé, la valeur doit être dans Notes, pas dans la cellule.
- **Step 2bis appliqué** : chaque ligne du CSV est une root cause actionnable, pas un symptôme isolé. Si Step 2bis a consolidé des symptômes, Notes mentionne explicitement cette consolidation.
- **Col 26 Dépend de** : si le use case a une dépendance inter-use-cases, elle est listée. Si la valeur référence un autre Use Case (col 1), vérifier que le nom matche exactement (lookup possible).
