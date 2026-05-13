# Règles de calcul use-case-value v1.2

Ce document détaille les formules, standards de méthode, formule de Score, et algorithme Verdict du CSV 27 colonnes.

## Règle d'or adoucie sur les estimations

**Cellules Impact $ (colonnes 13 à 18) acceptent** :

1. **Valeur citée verbatim** par un sponsor, un participant atelier ou une table source.

2. **Formule simple sur chiffres durs cités**, avec standards de méthode documentés comme paramètres de conversion :
   - Coût horaire fully-loaded : 80 $/h
   - FTE chargé fully-loaded : 100 000 $/an

3. **Inférence obvious sur un signal cité explicitement** dans les inputs, avec un standard de méthode documenté comme anchor de conversion. Exemples acceptés :
   - "1 personne à temps plein", "1 FTE saturé", "il y a quelqu'un dédié" → 100 000 $/an (FTE chargé standard) dans la cellule appropriée
   - "X h/sem" cité → X × 50 × 80 $/h dans la cellule appropriée
   - "5 personnes y consacrent une heure par jour" → 5 × 1 × 250 jours × 80 $/h dans la cellule appropriée

**Cellules Impact $ refusent toujours (gonflage interdit)** :
- **Multiplicateurs sortis du chapeau** : % captable, probabilité de perte, valeur moyenne contrat, taux de conversion, sans citation sponsor explicite
- **Volumes inventés** sans signal dans les inputs ("on perd vraisemblablement 5 contrats/an" si le sponsor n'a jamais cité de volume)
- **Extrapolations** "à valeur moyenne X $ par contrat" si la valeur n'a jamais été citée
- Toute valeur d'anchor inventée par le LLM

**Test de décision** : "Si on me demande d'où vient cette valeur de cellule, je peux pointer à une citation verbatim sponsor + appliquer un standard de méthode documenté → cellule OK. Si je dois inventer un paramètre de conversion → cellule = 0, l'analyse va en Notes (col 25) avec section À chiffrer en atelier."

**Une cellule à 0 ne signifie pas "pas d'impact"**. Elle signifie "non chiffré dans les inputs OU paramètres de conversion non disponibles". Toute cellule à 0 doit avoir une question correspondante structurée dans Notes (col 25), et son estimation indicative sera comptée dans col 27 Impact Potentiel Estimé.

## Standards de méthode — paramètres de conversion conditionnels

**Important** : ces deux standards sont des **paramètres de conversion qui s'appliquent UNIQUEMENT quand un signal verbal correspondant est cité dans les inputs**. Ce ne sont pas des valeurs appliquées automatiquement à toutes les lignes. Sans citation explicite du signal correspondant (heures par semaine, FTE saturé, etc.), la cellule reste à 0 et l'analyse va en Notes.

### Coût horaire fully-loaded standard de conversion

`80 $/h` — profil mixte employé québécois, charges sociales et overhead inclus.

**Quand l'appliquer** : uniquement si un sponsor cite des heures explicites ("20 h/sem", "6 h par dossier", "2 h/jour", etc.).

**Comment l'appliquer** :
```
Impact $ Temps Perdu = heures par semaine citées × 50 semaines × 80 $/h
```

Sans heures citées → cellule = 0, question "à chiffrer" en Notes.

Si le sponsor cite un taux différent (ex : "nos estimateurs sont à 90 $/h chargé"), utiliser ce taux et documenter la source dans Notes.

### FTE chargé fully-loaded standard de conversion

`100 000 $/an` — profil mixte employé québécois, charges sociales et overhead inclus, productivité annuelle effective.

**Quand l'appliquer** : uniquement si un sponsor cite explicitement une saturation FTE ("1 personne à temps plein", "1 FTE saturé", "Sophie est dédiée à ça", "il faut quelqu'un à plein temps"). La citation doit nommer une quantité de personne ou FTE.

**Comment l'appliquer** :
```
Impact $ Main d'Œuvre Saturation = nombre de FTE saturés cités × 100 000 $
OU (si la saturation se manifeste comme du temps consommé)
Impact $ Temps Perdu = équivalent FTE consommé sur tâche × 100 000 $
```

Sans citation FTE explicite → cellule = 0, question "à chiffrer" en Notes.

**Exemples de citations valides** : "Sophie est à temps plein sur les soumissions", "1 personne dédiée au matching PO", "Valérie est complètement saturée", "il faut quelqu'un de full-time là-dessus".

**Exemples de citations INVALIDES** (cellule = 0, va en Notes) : "ça mobilise du monde", "il y a beaucoup de travail", "c'est lourd", "ça pèse sur les équipes" — pas de quantité FTE nommée, donc pas d'anchor utilisable.

Si le sponsor cite un profil spécifique chargé différemment (ex : "expert principal à 150 000 $ chargé"), utiliser cette valeur et documenter la source dans Notes.

**Cohérence des deux standards** : 80 $/h × 1250 heures productives effectives par an ≈ 100 000 $. Les deux paramètres convergent quand le sponsor cite à la fois des heures et un FTE pour le même use case.

## Formules pour les colonnes calculées

### Col 19 — Impact Total Documenté ($/an)

```
Impact Total Documenté = SUM(col13 ; col14 ; col15 ; col16 ; col17 ; col18)
```

Reflète uniquement les chiffres durs et inférences obvious admissibles selon la règle d'or adoucie. Pas d'estimations Notes.

### Col 27 — Impact Potentiel Estimé ($/an)

Le LLM remplit cette colonne en sommant les estimations indicatives présentes dans Notes (col 25) du même use case, sous la section "À chiffrer en atelier".

Chaque "À titre indicatif, si [hypothèse], cela représenterait environ [valeur]" contribue sa valeur indicative à col 27.

Exemples :
- Notes mentionnent `À titre indicatif si 5 contrats × 100 000 $ × 30 % = +150 000 $` ET `Si 0.5 FTE évité ≈ +50 000 $` → col 27 = 200 000 $
- Notes ne contiennent aucune estimation indicative chiffrée → col 27 = 0

**Cette colonne ne remplace PAS Impact Total Documenté** (col 19). Elle s'ajoute uniquement pour le calcul du Score Priorité Impact, avec un discount d'incertitude (voir col 23).

### Col 23 — Score Priorité Impact (formule v1.2)

```
Score brut = (Impact Total Documenté + 0.3 × Impact Potentiel Estimé)
           × Complétude Chiffrage (extraite du label, valeur 1, 2 ou 3)
           × Urgence Stratégique (extraite du label, valeur 1, 2 ou 3)
           × (1 + 0.2 × log(MAX(1 ; Personnes Affectées)))

Score Priorité Impact (normalisé) =
   Score brut / MAX(Score brut sur toutes les lignes du CSV)
```

Le facteur **0.3** reflète la pénalité d'incertitude des estimations Notes vs chiffres durs. Un Impact Potentiel de 1 000 000 $ contribue donc 300 000 $ équivalent-documenté au Score.

**Logique** :
- Impact Total Documenté reste le levier principal (poids 1.0)
- Impact Potentiel Estimé est intégré avec discount (poids 0.3) pour éviter que les use cases haut potentiel non chiffré apparaissent à Score 0 dans le tri
- Complétude Chiffrage récompense les use cases bien documentés
- Urgence Stratégique applique le Cost of Delay (WSJF)
- Personnes Affectées en logarithme

### Col 24 — Verdict Impact (algorithme déterministe v1.2)

**Exécuter l'algorithme strictement séquentiellement, premier match l'emporte** :

```
1. SI Impact Total Documenté > 200 000 $
       ET Urgence Stratégique ≥ 2
       ET Complétude Chiffrage ≥ 2
   ALORS Verdict = "Critique"

2. SINON SI Impact Total Documenté > 100 000 $
            ET Complétude Chiffrage ≥ 2
   ALORS Verdict = "Forte"

3. SINON SI Impact Total Documenté > 100 000 $
            ET Complétude Chiffrage = 1
            ET au moins une cellule des col 13 à 18 > 0
   ALORS Verdict = "Forte (chiffrage à compléter)"

4. SINON SI Impact Total Documenté ≤ 100 000 $
            ET Impact Potentiel Estimé > 100 000 $
            ET Urgence Stratégique ≥ 2
            ET sponsor actif détecté dans Notes
   ALORS Verdict = "À chiffrer prioritairement"

5. SINON SI Impact Total Documenté ≤ 100 000 $
            ET Impact Potentiel Estimé > 0
   ALORS Verdict = "À chiffrer secondairement"

6. SINON SI Urgence Stratégique = 3
            ET Impact Total Documenté ≤ 100 000 $
   ALORS Verdict = "Stratégique long-terme"

7. SINON
   ALORS Verdict = "Faible impact"
```

**Détection "sponsor actif"** : le sponsor est cité dans les inputs avec un verbe d'action ("je pousse", "j'ai besoin", "c'est moi qui en parle", "cela me bloque", "ça m'excite", "il faut qu'on fasse ça"), ou un OKR explicite, ou une date butoir mentionnée.

**Anti-prudence-excessive** : si l'algorithme demande Verdict = "À chiffrer prioritairement" (étape 4), le LLM doit l'écrire tel quel. Ne pas downgrader en "À chiffrer secondairement" par prudence excessive. La condition est mécanique, pas d'interprétation.

Les seuils 200 000 $, 100 000 $ et le facteur 0.3 sont calibrés sur l'expérience MIA Innovation auprès de clients PME québécois.

## Lignes FORMULES et SOURCE (rows 2 et 3 du CSV)

Le gabarit `csv_template.csv` contient :

- **Row 1** : header avec les 27 noms de colonnes
- **Row 2 (FORMULES)** : pour chaque colonne calculée, la formule. Pour les colonnes data, "input direct" ou "manuel selon rubrique" ou "LLM dérive de Notes"
- **Row 3 (SOURCE)** : référence méthodologique (BABOK §10.10, WSJF, DAMA-DMBOK §3, 5 Whys + Ishikawa, MIA pratique terrain, input atelier)
- **Row 4+** : data, une ligne par use case

Ne jamais modifier les rows 2 et 3 lors du remplissage.

## Col 26 — Dépend de (texte libre, n'entre PAS dans le calcul)

La colonne 26 "Dépend de" capture les use cases prérequis identifiés en Step 2bis. Elle **n'entre pas dans la formule** du Score Priorité Impact ni dans l'algorithme Verdict. Signal qualitatif pour la synthèse (section "Dépendances et root causes") et la planification.

Format texte libre, nom du use case prérequis (matchant exactement la col 1 d'une autre ligne du CSV pour lookup mécanique), virgules pour séparer.

Exemples :
- Vide (cas par défaut)
- `Data Lake`
- `Data Lake, Workestimity opérationnel`

Les dépendances non-use-case (intégration tierce externe, condition contractuelle) vont en Notes (col 25).

## Vérifications de cohérence avant `present_files`

- Si col 19 (Impact Total Documenté) > 0 : au moins une cellule des col 13 à 18 doit être > 0 ET tracée à une citation source ou un standard de méthode appliqué à un signal cité (documenté dans Notes col 25).
- Si une cellule des col 13 à 18 est à 0 : Notes doit contenir une question "À chiffrer" correspondante avec estimation indicative chiffrée, qui contribue à col 27.
- Col 27 (Impact Potentiel Estimé) = somme stricte des estimations indicatives chiffrées présentes dans Notes du même use case. Vérifier la traçabilité ligne par ligne.
- Si Verdict = "Critique" : col 19 > 200 000 $ ET cellules > 0 toutes tracées dans Notes.
- Si Verdict = "À chiffrer prioritairement" : col 27 > 100 000 $ ET sponsor actif documenté dans Notes.
- Si Verdict = "À chiffrer secondairement" mais col 27 > 100 000 $ : c'est probablement une erreur d'application LLM par excès de prudence, repasser à "À chiffrer prioritairement" si conditions étape 4 vérifiées.
- Aucune cellule Impact $ ne contient une valeur sans anchor verbal (citation sponsor ou table source) + standard de méthode documenté.
- **Step 2bis appliqué** : chaque ligne est une root cause actionnable, pas un symptôme isolé.
- **Col 26 Dépend de** : si dépendance inter-use-cases, listée avec nom exact d'un autre Use Case du CSV.
