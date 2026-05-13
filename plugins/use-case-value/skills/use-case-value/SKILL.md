---
name: use-case-value
description: Priorise et chiffre la VALEUR business des use cases d'AI/automatisation à partir de tout matériel de découverte de processus (transcripts d'ateliers, post-its, inventaires CSV de 500+ lignes, notes de réunion, mixed). Six sources d'Impact $ explicites (temps perdu, opportunités manquées, coût des erreurs, saturation main-d'œuvre, dépendances externes, frictions inter-départementales) + nombre de personnes affectées + transversalité + pain points structurés. Règle d'or stricte : seuls les chiffres durs cités ou validés par le sponsor entrent dans les cellules Impact $, les estimations LLM vont exclusivement en Notes comme questions à poser au sponsor (à chiffrer en atelier). Aucune dimension d'effort technique (Build, Run, Payback, ROI An 1) : ce skill se concentre sur la VALEUR uniquement, l'effort sera traité par un futur skill séparé qui se branchera sur le CSV de sortie. Sortie : un dossier projet par run contenant un CSV 25 colonnes (avec FORMULES et SOURCE pour traçabilité) et une synthèse exécutive one-pager en prose narrative française Québec pour directeurs de compte. Triggers : prioriser, prioriser par impact, valeur use case, quel use case en premier, chiffrer la valeur, note de cadrage, atelier découverte, post-its, transcript atelier, inventaire processus, impact business, pertes évitées, opportunités manquées, saturation ressources, frictions inter-départementales.
---

# Use Case Value — Priorisation par impact chiffré

Ce skill transforme du matériel d'atelier de découverte en une **note de cadrage business** qui répond à une seule question : **"Quel use case mérite d'être chiffré et scopé en premier sur la base de l'impact business documenté ?"**

## Ce que ce skill produit

Un dossier `/mnt/user-data/outputs/priorisation_impact_<projet>_<YYYY-MM-DD>/` contenant :

1. **`priorisation_use_cases.csv`** : 25 colonnes, ligne FORMULES (row 2), ligne SOURCE (row 3), puis une ligne par use case. Le fichier de travail pour l'analyste.
2. **`synthese_priorisation.md`** : synthèse exécutive one-pager en prose narrative française Québec. Préambule + Top 5 + "À chiffrer en atelier" + "Recommandations méthodologiques".

## Ce que ce skill NE produit PAS

- Aucune estimation de Build, Run cost, Payback, ROI An 1, T-shirt size. L'effort technique est hors scope et sera traité par un futur skill complémentaire.
- Aucune estimation LLM dans les cellules Impact $. Voir règle d'or absolue ci-dessous.
- Aucun bundling de use cases dans la synthèse (un Top = un use case, pas un groupe).
- Aucune section Méthodologie longue ni Lectures transversales séparée.

## Toujours commencer par lire le framework

Avant de traiter tout input, lire dans cet ordre :

1. `references/framework.md` : explication des 25 colonnes, rubriques 1-3, sources méthodologiques (BABOK, WSJF, DAMA-DMBOK).
2. `references/calculation_rules.md` : formules, défaut coût horaire 80 $/h, grille de verdict.
3. `references/exemples_chiffrage_impact.md` : frontière chiffre dur vs hypothèse, exemples typiques colonne par colonne.

Sans ce contexte, tu vas remplir le CSV de manière incorrecte (notamment violer la règle d'or).

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

Annoncer le nom inféré à l'utilisateur. Si le projet est ambigu, demander confirmation rapide. L'utilisateur peut renommer le dossier après coup.

Créer le dossier de sortie : `/mnt/user-data/outputs/priorisation_impact_<projet>_<YYYY-MM-DD>/`.

### Step 1 — Inventorier les use cases

Pour chaque source d'input :

- Un use case est un **concept centré** (process, workflow, pain point central)
- Dans les post-its : les notes dominantes ou centrales du diagramme, généralement avec causes et conséquences pointant vers elles
- Dans les transcripts : les sujets discutés explicitement pour automatisation, généralement avec un sponsor identifiable
- Dans les tables d'inventaire : généralement une ligne par use case (vérifier d'abord le sens des colonnes)

Créer une ligne CSV par use case. Ne pas dupliquer.

Si deux sources décrivent le même process, **fusionner** les informations dans une seule ligne. Quand les sources sont en conflit sur une cellule, utiliser la plus crédible (transcript validé > note d'atelier > photo de post-it) et documenter le conflit dans Notes.

### Step 2 — Remplir colonnes 1 à 12 (identification, contexte, pain points)

- **Cols 1-5 Identification** : Use Case (titre court), Description (1-2 phrases), Sponsor (prénom + rôle ou département), Département principal, Type Agent (catégoriser : email auto / OCR / dashboard genAI / RAG / vision / RPA / autre).
- **Cols 6-9 Contexte** : Volume Annuel Qté + Volume Annuel Unité, Personnes Affectées (Nb), Transversalité (1-3 selon framework.md §Bloc 2).
- **Cols 10-12 Pain points** : 1 à 3 pain points listés brièvement (format `[Acteur] [verbe d'action] [contrainte ou perte]`, 1 phrase chacun). Pain Point Principal obligatoire ; #2 et #3 optionnels.

**Ne jamais inventer.** Si une cellule n'a pas de source claire dans l'input, la laisser vide.

### Step 3 — Chiffrer l'Impact $ (règle stricte)

Pour chaque colonne Impact $ (cols 13 à 18), procéder ligne par ligne :

1. **Parcourir les inputs** en quête de chiffres durs cités par un sponsor, un participant ou une table source pour cette dimension d'impact.

2. **Si un chiffre dur existe** ou si une formule simple s'applique (heures par semaine citées × 50 sem × coût horaire fully-loaded 80 $/h par défaut) :
   - Inscrire la valeur calculée dans la cellule
   - Documenter la citation source dans Notes (ex : `Sources : Shirley a dit "20 à 30 heures par semaine" (atelier 13 mai)`)
   - Documenter la formule dans Notes (ex : `Hypothèses : 25 h/sem × 50 sem × 80 $/h = 100 000 $`)

3. **Si pas de chiffre dur disponible** :
   - Laisser la cellule à **0**
   - Ajouter dans Notes une ligne sous la section "À chiffrer en atelier" :
     ```
     À chiffrer : [question précise au sponsor]. À titre indicatif, si
     [hypothèse réaliste], cela représenterait environ [valeur] en Impact $
     [colonne].
     ```
   - L'hypothèse doit être prudente, citer ses dérivations, et clairement présentée comme question à poser au sponsor, pas comme donnée.

4. **Ne jamais inscrire une estimation LLM dans une cellule Impact $.** Quand en doute, la cellule reste à 0 et la question va en Notes.

Consulter `references/exemples_chiffrage_impact.md` pour des exemples colonne par colonne (Temps Perdu, Opportunités Manquées, Coût Erreurs, Main d'Œuvre, Dépendances Externes, Frictions Inter-Départementales).

### Step 4 — Évaluer les scores 1-3 (cols 20 à 22)

- **Disponibilité Données (col 20)** selon rubrique framework.md §Bloc 5 :
  - `3 - structurées dans système accessible` : données déjà dans Business Central, CRM, fichier Excel structuré accessible
  - `2 - mixtes partiellement accessibles` : une partie dans système, l'autre dispersée
  - `1 - dispersées non structurées` : papier, post-it, mémoire des employés

- **Complétude Chiffrage (col 21)** reflète la richesse réelle des chiffres durs en cols 13-18, pas la richesse des hypothèses en Notes :
  - `3 - toutes sources pertinentes chiffrées` : 4 à 6 cellules sur 6 ont une valeur non-zéro tirée d'un chiffre dur
  - `2 - sources principales chiffrées, secondaires manquantes` : 2 à 3 cellules chiffrées, le reste à 0 avec questions en Notes
  - `1 - chiffres partiels ou hypothétiques en Notes uniquement` : 0 à 1 cellule chiffrée, l'essentiel est en hypothèse en Notes

- **Urgence Stratégique (col 22)** selon rubrique framework.md §Bloc 5 :
  - `3 - critique court terme` : OKR explicite client, fenêtre commerciale, sponsor pousse activement avec date butoir
  - `2 - important moyen terme` : sponsor identifié, mention récurrente, pas de date butoir
  - `1 - nice-to-have` : mention isolée, pas de sponsor actif

### Step 5 — Calculer Impact Total, Score Priorité, Verdict (cols 19, 23, 24)

Appliquer les formules de `references/calculation_rules.md` :

- **Col 19 Impact Total Documenté** = SUM(col13 à col18)
- **Col 23 Score Priorité Impact** = Impact Total × Complétude × Urgence × (1 + 0.2 × log(max(1, Personnes Affectées))), normalisé sur le max du CSV
- **Col 24 Verdict Impact** = lookup sur la grille (Critique / Forte / À chiffrer prioritairement / À chiffrer secondairement / Stratégique long-terme / Faible impact)

### Step 6 — Rédiger la synthèse en prose narrative

Suivre le gabarit `references/synthese_template.md`. Règles clés :

- **Préambule** (1 paragraphe) : date, sources d'inputs, nombre de use cases analysés, rappel de la règle d'or chiffres durs vs hypothèses, scope impact-only.
- **5 Top en prose narrative** (5 à 8 lignes chacun) : sans pipe, sans crochet, sans em dash, sans `---` entre Top. Sponsor sur une ligne, situation et chiffres durs en prose, impact documenté ventilé, verdict intégré dans la phrase, ouverture sur les questions à chiffrer si pertinent.
- **Section "À chiffrer en atelier avant décision"** : use cases avec Verdict "À chiffrer prioritairement" ou "Stratégique long-terme" pas déjà dans le Top 5, avec questions clés à poser.
- **Section "Recommandations méthodologiques"** : 1-2 paragraphes sur les limites du chiffrage actuel et la prochaine étape (atelier de validation, futur skill effort).

**Style français Québec strict** :
- Aucun em dash dans tout le document (remplacer par virgule, parenthèse, deux-points).
- Chiffres en toutes lettres : `220 000 $` pas `220k$`, `5 mois` pas `5m`.
- Garder uniquement les symboles `%` et `$`.

Sauvegarder dans `priorisation_use_cases.csv` et `synthese_priorisation.md` du dossier projet (Step 0).

### Step 7 — Présenter le dossier

Utiliser `present_files` pour partager les deux fichiers. CSV en premier (artefact de travail principal), puis synthèse markdown.

## Principes critiques

**Ne jamais inventer de données quantitatives.** Les cellules vides ou à 0 sont valables et utiles : elles signalent ce qui reste à chiffrer en atelier.

**La cellule à 0 sans question en Notes est une erreur.** Si tu mets 0, tu DOIS ajouter une question structurée dans Notes.

**Ne jamais inscrire une estimation LLM dans une cellule Impact $.** Quand en doute : 0 dans la cellule, hypothèse en Notes.

**Ne pas surcharger le verdict.** Si la math dit "À chiffrer prioritairement", ne pas écrire "Critique" parce que tu as une intuition. L'intuition va dans Notes.

**Sortie en français Québec.** Même si les inputs sont en anglais. Aucun em dash.

## Cas particuliers

- **Pas de données quantitatives du tout** : remplir cols 1-12 (identification et pain points). Toutes les cols 13-18 à 0 avec questions structurées en Notes. Complétude Chiffrage = 1. La synthèse recommande explicitement un atelier chiffrage.

- **Un seul use case** : générer un CSV 1 ligne. La synthèse se concentre sur ce use case avec analyse approfondie en prose narrative.

- **500+ use cases dans un inventaire** : traiter toutes les lignes. Le CSV gère. La synthèse garde le Top 5. Si traitement trop long, batcher par département ou par Urgence Stratégique et alerter l'utilisateur.

- **Information conflictuelle entre sources** : documenter le conflit dans Notes. Utiliser la source la plus crédible pour la cellule (transcript validé > note d'atelier > photo de post-it).

- **Inputs en anglais** : traduire à la sortie. Le CSV et la synthèse sont toujours en français Québec.

## Quality checks avant `present_files`

- [ ] Dossier de sortie créé avec nom `priorisation_impact_<projet>_<YYYY-MM-DD>/`
- [ ] CSV a la ligne FORMULES (row 2) et SOURCE (row 3) préservées
- [ ] Chaque ligne data a au moins Use Case et Sponsor remplis
- [ ] **Aucune cellule Impact $ ne contient une estimation LLM** : toutes les valeurs > 0 ont une citation source ou une formule simple documentée dans Notes
- [ ] Les cellules à 0 ont une question structurée correspondante dans Notes sous "À chiffrer en atelier"
- [ ] Verdict matche la grille (cf. calculation_rules.md §Col 24)
- [ ] Aucun em dash dans CSV ni dans synthèse
- [ ] Synthèse a préambule + Top 5 + À chiffrer en atelier + Recommandations méthodologiques
- [ ] Synthèse en prose narrative complète : phrases complètes, pas de fragments à pipe, pas de crochets, pas de séparateurs `---` entre Top
- [ ] Chiffres en toutes lettres dans la synthèse (220 000 $ pas 220k$)
