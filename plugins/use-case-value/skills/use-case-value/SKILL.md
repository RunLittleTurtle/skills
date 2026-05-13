---
name: use-case-value
description: Priorise et chiffre la VALEUR business des use cases d'AI/automatisation à partir de tout matériel de découverte de processus (transcripts d'ateliers, post-its, inventaires CSV de 500+ lignes, notes de réunion, mixed). Six sources d'Impact $ explicites (temps perdu, opportunités manquées, coût des erreurs, saturation main-d'œuvre, dépendances externes, frictions inter-départementales) + nombre de personnes affectées + transversalité + pain points structurés + colonne Dépend de pour modéliser les dépendances inter-use-cases. Step 2bis Root Cause Analysis (5 Whys + Ishikawa) appliquée avant chiffrage pour distinguer root causes actionnables vs symptômes (évite de triple-compter l'impact). Règle d'or stricte : seuls les chiffres durs cités ou validés par le sponsor entrent dans les cellules Impact $, les estimations LLM vont exclusivement en Notes comme questions à poser au sponsor (à chiffrer en atelier). Aucune dimension d'effort technique (Build, Run, Payback, ROI An 1) : ce skill se concentre sur la VALEUR uniquement, l'effort sera traité par un futur skill séparé qui se branchera sur le CSV de sortie. Sortie : un dossier projet par run contenant un CSV 26 colonnes (avec FORMULES et SOURCE pour traçabilité) et une synthèse exécutive one-pager en prose narrative française Québec, avec section optionnelle Dépendances et root causes si pertinent. Triggers : prioriser, prioriser par impact, valeur use case, quel use case en premier, chiffrer la valeur, note de cadrage, atelier découverte, post-its, transcript atelier, inventaire processus, impact business, root cause, dépendances use cases, 5 whys, ishikawa.
---

# Use Case Value — Priorisation par impact chiffré

Ce skill transforme du matériel d'atelier de découverte en une **note de cadrage business** qui répond à une seule question : **"Quel use case mérite d'être chiffré et scopé en premier sur la base de l'impact business documenté ?"**

## Ce que ce skill produit

Un dossier `/mnt/user-data/outputs/priorisation_impact_<projet>_<YYYY-MM-DD>/` contenant :

1. **`priorisation_use_cases.csv`** : 26 colonnes, ligne FORMULES (row 2), ligne SOURCE (row 3), puis une ligne par use case. Le fichier de travail pour l'analyste.
2. **`synthese_priorisation.md`** : synthèse exécutive one-pager en prose narrative française Québec. Préambule + Top 5 + "À chiffrer en atelier" + "Dépendances et root causes" (section optionnelle) + "Recommandations méthodologiques".

## Ce que ce skill NE produit PAS

- Aucune estimation de Build, Run cost, Payback, ROI An 1, T-shirt size. L'effort technique est hors scope et sera traité par un futur skill complémentaire.
- Aucune estimation LLM dans les cellules Impact $. Voir règle d'or absolue ci-dessous.
- Aucun bundling de use cases dans la synthèse (un Top = un use case actionnable, les dépendances entre eux sont décrites dans la section dédiée).
- Aucune section Méthodologie longue.

## Toujours commencer par lire le framework

Avant de traiter tout input, lire dans cet ordre :

1. `references/framework.md` : explication des 26 colonnes, rubriques 1-3, sources méthodologiques (BABOK, WSJF, 5 Whys + Ishikawa, DAMA-DMBOK).
2. `references/calculation_rules.md` : formules, défaut coût horaire 80 $/h, grille de verdict.
3. `references/exemples_chiffrage_impact.md` : frontière chiffre dur vs hypothèse, exemples typiques colonne par colonne.

Sans ce contexte, tu vas remplir le CSV de manière incorrecte (notamment violer la règle d'or ou lister des symptômes au lieu de root causes).

## Règle d'or absolue : chiffres durs vs estimations LLM

**Cellules Impact $ (colonnes 13 à 18)** acceptent uniquement :
- Une valeur citée explicitement par un sponsor ou un participant atelier (citation verbatim disponible)
- Une valeur dérivée par formule simple sur des chiffres durs (ex : "20-30 heures par semaine" × 50 sem × 80 $/h fully-loaded standard)
- Sinon : **0**

**Colonne Notes (col 25)** est l'endroit où les estimations indicatives vont, sous forme structurée :

```
À chiffrer en atelier :
- [Question précise au sponsor]. À titre indicatif, si [hypothèse réaliste],
  cela représenterait environ [valeur] de plus en Impact $ [colonne], soit
  [+X %] d'Impact Total.
```

**Une cellule Impact $ à 0 ne signifie pas "pas d'impact"**. Elle signifie "non chiffré dans les inputs". Le Verdict (col 24) gère cette nuance : un use case avec Impact Total Documenté faible mais potentiel élevé en Notes peut classer "À chiffrer prioritairement".

**Une cellule à 0 sans question correspondante en Notes est une erreur de remplissage.** Si tu mets 0, tu DOIS ajouter une question "À chiffrer" dans Notes.

## Inputs acceptés

- Photos de post-its ou de whiteboards d'ateliers de découverte
- Transcripts de réunions stakeholders ou architectes (toute langue, sortie en français Québec)
- Tableaux d'inventaire de processus (Excel ou CSV, jusqu'à 500+ lignes)
- Notes libres d'atelier ou de réunion
- Mix de tous ces formats

Si aucun input n'est fourni, demander à l'utilisateur ce qu'il a comme matériel. Ne jamais inventer de use cases.

## Workflow

### Step 0 — Détecter le projet et créer le dossier de sortie

Inférer le nom du projet à partir des inputs, dans cet ordre de priorité :

1. Nom du fichier source (ex : `faspac_atelier.md` → projet = `faspac`)
2. Entité client mentionnée en boucle dans le transcript ou la note
3. Sinon : fallback `<YYYY-MM-DD>_<HHMM>` (date-heure du moment)

Annoncer le nom inféré à l'utilisateur. Si le projet est ambigu, demander confirmation rapide.

Créer le dossier de sortie : `/mnt/user-data/outputs/priorisation_impact_<projet>_<YYYY-MM-DD>/`.

### Step 1 — Inventorier les candidats use cases bruts

Parcourir tous les inputs et lister tous les **candidats use cases bruts** : tout concept centré qui semble être un process, un workflow ou un pain point automatisable.

À ce stade, **ne pas filtrer**. On peut avoir 15-20 candidats incluant des doublons et des symptômes.

### Step 2bis — Analyse root cause (mini 5 Whys + Ishikawa léger)

**Avant de chiffrer l'impact**, identifier pour chaque candidat brut s'il est une **root cause** (cause profonde actionnable par une automatisation) ou un **symptôme** (effet visible d'un autre process qui en serait la vraie cause).

Pour chaque candidat, appliquer un mini 5 Whys :

1. Quel est le pain point apparent ? (ex : "Valérie est saturée par les soumissions")
2. Pourquoi ? (ex : "Le process d'estimation manuel from Business Central est lent")
3. Pourquoi ce process est-il lent ? (ex : "Pas d'agent qui pull les données BC, calcule la marge et drafte le doc")
4. Pourquoi pas d'agent ? (ex : "Pas encore construit, c'est précisément le use case d'automatisation potentiel")

Le **use case à enregistrer dans le CSV est celui qui est actionnable par une automatisation** (le niveau le plus profond avant qu'on quitte le périmètre IA/automatisation). Dans l'exemple ci-dessus : `Agent d'estimation soumission from BC`, pas `Valérie saturée`.

**Comment représenter dans le CSV** :

- **Plusieurs symptômes, une seule automatisation possible** : créer **une seule ligne** pour la root cause. Les symptômes individuels sont listés en Pain Points #1, #2, #3 (cols 10-12). Notes mentionne explicitement `Root cause : cette automatisation est la root cause des symptômes [X], [Y], [Z] consolidés ici en Step 2bis pour éviter de double-compter l'impact.`

- **Symptômes assez distincts pour scopes séparés mais partageant une dépendance** (ex : 3 use cases distincts qui ont tous besoin du même Data Lake) : créer **plusieurs lignes**, chacune avec col 26 "Dépend de" pointant vers la fondation partagée.

- **Le concept partagé est lui-même un use case d'infrastructure** (ex : Data Lake, intégration BC, plateforme de connecteurs) : créer **une ligne pour la fondation** + plusieurs lignes pour les use cases consommateurs qui ont col 26 "Dépend de" = nom de la fondation. La fondation aura souvent un Impact Total Documenté faible et un Verdict "Stratégique long-terme" mécanique, mais la section synthèse "Dépendances et root causes" l'élèvera comme priorité de séquence.

**Pourquoi cette Step est critique** : sans elle, le CSV liste 3-5 symptômes en parallèle (saturation, lenteur, erreurs, double-saisie, friction) et triple-compte l'impact d'une seule automatisation possible. Le LLM doit comprimer les candidats bruts (Step 1) en root causes actionnables (sortie Step 2bis) avant de remplir les cols 6-25.

À la fin de Step 2bis, tu as une liste réduite et nette : 5 à 12 root causes actionnables là où tu avais 15-20 candidats bruts.

### Step 2 — Remplir colonnes 1 à 12 (identification, contexte, pain points)

Pour chaque root cause actionnable retenue après Step 2bis :

- **Cols 1-5 Identification** : Use Case (titre court), Description (1-2 phrases), Sponsor, Département principal, Type Agent.
- **Cols 6-9 Contexte** : Volume Annuel Qté + Volume Annuel Unité, Personnes Affectées, Transversalité (1-3).
- **Cols 10-12 Pain points** : 1 à 3 pain points listés brièvement. Si Step 2bis a consolidé plusieurs symptômes sous une root cause, les lister ici en Pain Points #1 à #3.

**Ne jamais inventer.** Si une cellule n'a pas de source claire dans l'input, la laisser vide.

### Step 3 — Chiffrer l'Impact $ (règle stricte)

Pour chaque colonne Impact $ (cols 13 à 18) :

1. **Parcourir les inputs** en quête de chiffres durs cités par un sponsor, un participant ou une table source pour cette dimension d'impact sur la **root cause** retenue (pas le symptôme).

2. **Si un chiffre dur existe** ou si une formule simple s'applique (heures par semaine citées × 50 sem × coût horaire fully-loaded 80 $/h par défaut) :
   - Inscrire la valeur calculée dans la cellule
   - Documenter la citation source dans Notes
   - Documenter la formule dans Notes

3. **Si pas de chiffre dur disponible** :
   - Laisser la cellule à **0**
   - Ajouter dans Notes une ligne sous "À chiffrer en atelier" avec question structurée et estimation indicative

4. **Ne jamais inscrire une estimation LLM dans une cellule Impact $.** Quand en doute, la cellule reste à 0 et la question va en Notes.

Consulter `references/exemples_chiffrage_impact.md` pour des exemples colonne par colonne.

### Step 4 — Évaluer les scores 1-3 (cols 20 à 22)

Selon les rubriques de `framework.md` §Bloc 5 :
- Disponibilité Données (col 20) : 3 / 2 / 1 selon accessibilité
- Complétude Chiffrage (col 21) : reflète la richesse des chiffres durs en 13-18, pas les hypothèses
- Urgence Stratégique (col 22) : Cost of Delay WSJF

### Step 5 — Calculer Impact Total, Score Priorité, Verdict (cols 19, 23, 24)

Appliquer les formules de `references/calculation_rules.md`.

### Step 5bis — Remplir Dépend de (col 26)

Pour chaque ligne, identifier les use cases **prérequis** identifiés en Step 2bis :

- Si la root cause a besoin qu'un autre use case du CSV soit livré en premier pour atteindre son Impact, lister le nom exact de l'autre Use Case (col 1) dans Dépend de
- Format texte libre, plusieurs prérequis séparés par virgules
- Vide si aucune dépendance externe au sein du périmètre IA (cas par défaut)
- Les dépendances **non-use-case** (intégration tierce, condition contractuelle, autre projet hors scope IA) vont en Notes, pas dans col 26

### Step 6 — Rédiger la synthèse en prose narrative

Suivre le gabarit `references/synthese_template.md`. Sections :

1. **Préambule** (1 paragraphe) : date, sources, nombre de use cases analysés (avant et après Step 2bis), rappel règle d'or et root cause, scope impact-only.
2. **Top 5 en prose narrative** (5 à 8 lignes chacun, sans pipe, sans crochet, sans em dash, sans `---` entre Top).
3. **À chiffrer en atelier avant décision** : use cases avec Verdict "À chiffrer prioritairement" ou "Stratégique long-terme" pas dans le Top 5.
4. **Dépendances et root causes** (section optionnelle) : à inclure uniquement si col 26 a des valeurs non vides OU si Step 2bis a consolidé des symptômes. 2-3 paragraphes prose.
5. **Recommandations méthodologiques** (1-2 paragraphes).

**Style français Québec strict** : aucun em dash, chiffres en toutes lettres (`220 000 $` pas `220k$`), garder uniquement `%` et `$` comme symboles.

### Step 7 — Présenter le dossier

Utiliser `present_files` pour partager les deux fichiers. CSV en premier, puis synthèse markdown.

## Principes critiques

**Step 2bis n'est pas optionnel.** Sans cette analyse, on liste des symptômes en parallèle et on triple-compte l'impact. Si en doute sur un candidat, appliquer 5 Whys.

**Ne jamais inventer de données quantitatives.** Cellules vides ou à 0 = ce qui reste à chiffrer en atelier.

**La cellule à 0 sans question en Notes est une erreur.** Si tu mets 0, tu DOIS ajouter une question structurée dans Notes.

**Ne jamais inscrire une estimation LLM dans une cellule Impact $.** Quand en doute : 0 dans la cellule, hypothèse en Notes.

**Ne pas surcharger le verdict.** L'intuition va dans Notes, pas dans le verdict.

**Sortie en français Québec.** Même si les inputs sont en anglais. Aucun em dash.

## Cas particuliers

- **Pas de données quantitatives du tout** : remplir cols 1-12 + Step 2bis. Toutes les cols 13-18 à 0 avec questions structurées en Notes. Complétude Chiffrage = 1. La synthèse recommande explicitement un atelier chiffrage.

- **Un seul use case** : générer un CSV 1 ligne. La synthèse se concentre sur ce use case en prose narrative approfondie.

- **500+ use cases dans un inventaire** : Step 2bis va probablement réduire à 50-100 root causes actionnables. Traiter toutes les lignes. La synthèse garde le Top 5. Batcher par département si trop long.

- **Information conflictuelle entre sources** : documenter dans Notes, utiliser la source la plus crédible (transcript validé > note d'atelier > photo de post-it).

- **Inputs en anglais** : traduire à la sortie. CSV et synthèse en français Québec.

## Quality checks avant `present_files`

- [ ] Dossier de sortie créé avec nom `priorisation_impact_<projet>_<YYYY-MM-DD>/`
- [ ] CSV a 26 colonnes, ligne FORMULES (row 2) et SOURCE (row 3) préservées
- [ ] **Step 2bis Root Cause appliqué** : chaque ligne est une root cause actionnable, pas un symptôme isolé. Notes documente la consolidation si applicable.
- [ ] Chaque ligne data a au moins Use Case et Sponsor remplis
- [ ] **Aucune cellule Impact $ ne contient une estimation LLM** : toutes les valeurs > 0 ont une citation source ou une formule simple documentée dans Notes
- [ ] Les cellules à 0 ont une question structurée correspondante dans Notes sous "À chiffrer en atelier"
- [ ] **Col 26 Dépend de remplie** pour les use cases avec dépendances inter-use-cases identifiées en Step 2bis
- [ ] Verdict matche la grille (cf. calculation_rules.md §Col 24)
- [ ] Aucun em dash dans CSV ni dans synthèse
- [ ] Synthèse a préambule + Top 5 + À chiffrer en atelier + Dépendances et root causes (si applicable) + Recommandations méthodologiques
- [ ] Synthèse en prose narrative : phrases complètes, pas de fragments à pipe, pas de crochets, pas de séparateurs `---` entre Top
- [ ] Chiffres en toutes lettres dans la synthèse
