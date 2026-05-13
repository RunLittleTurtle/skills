---
name: use-case-value
description: Priorise et chiffre la VALEUR business des use cases d'AI/automatisation à partir de tout matériel de découverte de processus (transcripts d'ateliers, post-its, inventaires CSV de 500+ lignes, notes de réunion). Six sources d'Impact $ explicites (temps perdu, opportunités manquées, coût des erreurs, saturation main-d'œuvre, dépendances externes, frictions inter-départementales) + pain points + personnes affectées + transversalité + colonne Dépend de pour modéliser les dépendances inter-use-cases + colonne Impact Potentiel Estimé pour intégrer les estimations Notes dans le Score avec discount. Step 2bis Root Cause Analysis (5 Whys + Ishikawa) appliquée avant chiffrage pour distinguer root causes actionnables vs symptômes. Règle d'or adoucie v1.2 : cellules acceptent les chiffres durs cités, les formules simples sur chiffres durs, ET les inférences obvious sur signaux cités via standards de méthode documentés (80 $/h pour heures citées, 100 000 $/FTE pour saturation FTE citée). Refusé : multiplicateurs sortis du chapeau (% captable, probabilité, valeur moyenne sans citation). Aucune dimension d'effort technique (Build, Run, Payback) : ce skill se concentre sur la VALEUR uniquement, l'effort sera traité par un futur skill séparé. Sortie : un dossier projet par run contenant un CSV 27 colonnes (avec FORMULES et SOURCE pour traçabilité) et une synthèse exécutive one-pager en prose narrative française Québec. Triggers : prioriser, valeur use case, chiffrer impact, note de cadrage, atelier découverte, post-its, transcript, root cause, dépendances use cases, 5 whys, ishikawa.
---

# Use Case Value v1.2 — Priorisation par impact chiffré

Ce skill transforme du matériel d'atelier de découverte en une **note de cadrage business** qui répond à une seule question : **"Quel use case mérite d'être chiffré et scopé en premier sur la base de l'impact business documenté ou clairement chiffrable ?"**

## Ce que ce skill produit

Un dossier `/mnt/user-data/outputs/priorisation_impact_<projet>_<YYYY-MM-DD>/` contenant :

1. **`priorisation_use_cases.csv`** : 27 colonnes, ligne FORMULES (row 2), ligne SOURCE (row 3), puis une ligne par use case.
2. **`synthese_priorisation.md`** : synthèse exécutive one-pager en prose narrative française Québec.

## Ce que ce skill NE produit PAS

- Aucune estimation de Build, Run cost, Payback, ROI An 1, T-shirt. L'effort technique est hors scope.
- Aucune valeur inventée dans les cellules Impact $. Voir règle d'or adoucie ci-dessous.
- Aucun bundling de use cases dans la synthèse (un Top = une root cause actionnable).

## Toujours commencer par lire le framework

Avant de traiter tout input, lire dans cet ordre :

1. `references/framework.md` : 27 colonnes, rubriques 1-3, sources méthodologiques.
2. `references/calculation_rules.md` : standards de méthode (80 $/h, 100 000 $/FTE), formule Score v1.2, algorithme Verdict déterministe.
3. `references/exemples_chiffrage_impact.md` : frontière chiffre dur / inférence obvious / hypothèse, exemples colonne par colonne.

Sans ce contexte, tu vas remplir le CSV de manière incorrecte (notamment violer la règle d'or ou lister des symptômes au lieu de root causes).

## Règle d'or adoucie v1.2

**Cellules Impact $ (colonnes 13 à 18) acceptent** :

1. **Valeur citée verbatim** par un sponsor ou participant atelier
2. **Formule simple sur chiffres durs cités**, avec standards de méthode comme paramètres de conversion :
   - 80 $/h (coût horaire fully-loaded standard) appliqué à des heures par semaine citées
   - 100 000 $/an (FTE chargé fully-loaded standard) appliqué à une saturation FTE citée
3. **Inférence obvious sur signal cité explicitement** :
   - "Sophie est à temps plein sur les soumissions" → col Main d'Œuvre = 100 000 $ (FTE standard appliqué à citation FTE)
   - "20 h/sem de matching" → col Temps Perdu = 80 000 $ (coût horaire standard appliqué à heures citées)
   - "5 personnes 1h/jour" → conversion directe avec heures × 250 × 80 $/h

**Cellules Impact $ refusent toujours (gonflage interdit)** :
- Multiplicateurs sortis du chapeau : % captable, probabilité de perte, valeur moyenne contrat sans citation
- Volumes inventés sans signal dans les inputs
- Extrapolations sans anchor verbal

**Test de décision** : "Pour cette valeur de cellule, est-ce que tous les paramètres de la formule sont (a) cités verbatim dans les inputs OU (b) un standard de méthode documenté (80 $/h, 100 000 $/FTE) ? Si OUI les deux → cellule. Si NON, paramètre inventé → cellule = 0, hypothèse en Notes."

**Une cellule à 0 ne signifie pas "pas d'impact"**. Elle signifie "non chiffré dans les inputs OU paramètres de conversion non disponibles". Toute cellule à 0 doit avoir une question correspondante structurée dans Notes (col 25), et son estimation indicative chiffrée alimentera col 27 Impact Potentiel Estimé.

## Inputs acceptés

- Photos de post-its / whiteboards
- Transcripts (toute langue, sortie en français Québec)
- Tableaux d'inventaire de processus (jusqu'à 500+ lignes)
- Notes libres
- Mix de ces formats

Si aucun input fourni : demander à l'utilisateur. Ne jamais inventer de use cases.

## Workflow

### Step 0 — Détecter le projet et créer le dossier de sortie

Inférer le nom du projet (nom de fichier source, entité client en boucle dans transcript, ou fallback date-heure).

Créer `/mnt/user-data/outputs/priorisation_impact_<projet>_<YYYY-MM-DD>/`.

### Step 1 — Inventorier les candidats use cases bruts

Lister tous les concepts centrés (process, workflow, pain point automatisable) qui apparaissent dans les inputs. Ne pas filtrer à ce stade.

### Step 2bis — Analyse Root Cause (mini 5 Whys + Ishikawa léger)

Avant de chiffrer, identifier pour chaque candidat brut s'il est une **root cause** (cause profonde actionnable) ou un **symptôme** (effet d'un autre process).

Pour chaque candidat, appliquer un mini 5 Whys :
1. Quel pain point apparent ?
2. Pourquoi ? Quelle cause sous-jacente ?
3. Pourquoi cette cause persiste-t-elle ?
4. Quel niveau d'automatisation l'adresserait directement ?

Le **use case à enregistrer** est celui actionnable par automatisation. Les symptômes consolidés deviennent Pain Points #1, #2, #3 sur la ligne de la root cause.

**Représentation CSV** :
- Plusieurs symptômes / une automatisation possible → une seule ligne pour la root cause, symptômes en Pain Points
- Symptômes distincts mais dépendance partagée → plusieurs lignes avec col 26 Dépend de pointant vers la fondation
- Le concept partagé est lui-même un use case (Data Lake, intégration) → ligne pour la fondation + lignes consommatrices avec col 26 pointant vers la fondation

Documenter dans Notes (col 25) : `Root cause : cette ligne consolide les symptômes X, Y, Z identifiés en Step 2bis pour éviter de double-compter.`

### Step 2 — Remplir colonnes 1 à 12

Pour chaque root cause retenue : Use Case, Description, Sponsor, Département, Type Agent, Volume, Personnes Affectées, Transversalité, Pain Points #1-3 (les symptômes consolidés si applicable).

Ne jamais inventer. Cellule sans source claire = vide.

### Step 3 — Chiffrer l'Impact $ (règle d'or adoucie v1.2)

Pour chaque colonne Impact $ (13 à 18) :

1. **Parcourir les inputs** pour chiffres durs cités ou signaux convertibles via standards (heures citées, FTE saturé cité, valeur explicite citée).

2. **Si chiffre dur direct OU formule simple sur citation OU inférence obvious sur signal cité** :
   - Inscrire la valeur dans la cellule
   - Documenter dans Notes la citation source + la formule de conversion utilisée
   - Exemple : `Sources : Sophie a dit "20 h/sem". Hypothèses : 20 × 50 × 80 $/h = 80 000 $.`
   - Exemple FTE : `Sources : Maxime a dit "Sophie est à temps plein". Hypothèses : 1 FTE × 100 000 $ standard = 100 000 $.`

3. **Si paramètre de conversion devrait être inventé** (% captable, probabilité, valeur unitaire sans citation) :
   - Laisser cellule à **0**
   - Ajouter dans Notes section "À chiffrer en atelier" :
     ```
     À chiffrer : [question]. À titre indicatif si [hypothèse réaliste], cela
     représenterait environ [valeur] en Impact $ [colonne].
     ```

4. **Calculer col 27 Impact Potentiel Estimé** à la fin de Step 3 :
   - Somme stricte des estimations indicatives chiffrées présentes dans Notes du même use case
   - Si Notes mentionne `+150 000 $ Opportunités` ET `+50 000 $ Main d'Œuvre` → col 27 = 200 000 $
   - Si aucune estimation indicative chiffrée → col 27 = 0

Consulter `references/exemples_chiffrage_impact.md` pour exemples colonne par colonne avec la règle d'or adoucie.

### Step 4 — Évaluer scores 1-3 (cols 20 à 22)

- Disponibilité Données (col 20) : DAMA-DMBOK Accessibility
- Complétude Chiffrage (col 21) : rubrique élargie v1.2 (cellule dominante chiffrée = niveau 2)
- Urgence Stratégique (col 22) : WSJF Cost of Delay

Voir `framework.md` §Bloc 5 pour rubriques précises.

### Step 5 — Calculer Impact Total, Score, Verdict (cols 19, 23, 24)

- **Col 19 Impact Total Documenté** = SUM(col13:col18)
- **Col 23 Score Priorité Impact** = `(col19 + 0.3 × col27) × Complétude × Urgence × (1 + 0.2 × log(Personnes))`, normalisé sur max
- **Col 24 Verdict Impact** = algorithme déterministe numéroté de `calculation_rules.md`

### Step 5bis — Remplir Dépend de (col 26)

Pour chaque ligne, lister les use cases prérequis identifiés en Step 2bis (texte libre, séparés par virgules, vide si aucune dépendance). Les dépendances non-use-case vont en Notes, pas en col 26.

### Step 6 — Rédiger la synthèse en prose narrative

Suivre `references/synthese_template.md`. Sections : Préambule + Top 5 + À chiffrer en atelier + Dépendances et root causes (optionnel) + Recommandations méthodologiques.

Style français Québec strict : pas d'em dash, pas de crochets, pas de pipes dans le corps, chiffres en toutes lettres (`220 000 $`), garder uniquement `%` et `$`.

### Step 7 — Présenter le dossier

Utiliser `present_files` : CSV d'abord, puis synthèse.

## Principes critiques

**Step 2bis n'est pas optionnel.** Sans cette analyse, on liste des symptômes et on triple-compte l'impact.

**Inférence obvious autorisée v1.2, gonflage toujours interdit.** Si le sponsor cite "1 FTE saturé", convertir avec le standard 100 000 $/an est légitime. Si le sponsor dit "ça mobilise du monde" sans nommer de quantité, cellule = 0.

**Ne pas surcharger le Verdict par prudence excessive.** L'algorithme est déterministe. Si l'étape 4 demande "À chiffrer prioritairement" parce que col 27 > 100 000 $ et sponsor actif détecté, écrire "À chiffrer prioritairement". Ne pas downgrader en "secondairement".

**Sortie en français Québec.** Aucun em dash.

## Cas particuliers

- **Pas de chiffres durs ni d'inférence obvious possible** : cols 13-18 à 0, col 27 capture le potentiel. Complétude = 1. Synthèse recommande atelier chiffrage.
- **Un seul use case** : CSV 1 ligne, synthèse approfondie.
- **500+ use cases** : Step 2bis va réduire à 50-100 root causes. CSV gère, synthèse garde Top 5.
- **Information conflictuelle** : documenter dans Notes, source la plus crédible l'emporte.

## Quality checks avant `present_files`

- [ ] Dossier `priorisation_impact_<projet>_<YYYY-MM-DD>/` créé
- [ ] CSV a 27 colonnes, lignes FORMULES (row 2) et SOURCE (row 3) préservées
- [ ] **Step 2bis Root Cause appliqué** : chaque ligne est une root cause actionnable
- [ ] Chaque ligne a au moins Use Case et Sponsor remplis
- [ ] **Aucune cellule Impact $ ne contient une valeur sans anchor** : toutes valeurs > 0 sont (a) une citation verbatim sponsor OU (b) une formule simple sur citation OU (c) une inférence obvious sur signal cité + standard de méthode documenté. Tracé dans Notes.
- [ ] Les cellules à 0 ont une question structurée correspondante dans Notes
- [ ] **Col 27 Impact Potentiel Estimé** = somme stricte des estimations chiffrées présentes dans Notes
- [ ] **Col 26 Dépend de** remplie si dépendances inter-use-cases
- [ ] Verdict matche l'algorithme déterministe (cf. calculation_rules.md)
- [ ] **Anti-prudence-excessive** : si col 27 > 100 000 $ ET sponsor actif ET Urgence ≥ 2, Verdict doit être "À chiffrer prioritairement", pas "secondairement"
- [ ] Aucun em dash dans CSV ni synthèse
- [ ] Synthèse a préambule + Top 5 + À chiffrer + Dépendances et root causes (si pertinent) + Recommandations
- [ ] Synthèse en prose narrative complète
- [ ] Chiffres en toutes lettres
