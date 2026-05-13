# Gabarit synthèse exécutive use-case-value v1.1

Ce gabarit produit une **note de cadrage one-pager en prose narrative française Québec** pour les directeurs de compte et sponsors clients.

## Règles de style absolues

- **Prose narrative**. Phrases complètes, pas de fragments à pipe ni de fragments télégraphiques.
- **Pas d'em dash** dans aucun texte. Remplacer par virgule, parenthèse ou deux-points.
- **Pas de crochets** comme `[XX %]` ou `[X mois]`. Écrire en toutes lettres.
- **Pas de séparateurs** `---` entre les Top. Une ligne vide suffit, les titres `### N.` font la séparation visuelle.
- **Pas de pipe** `|` dans le corps du texte (réservé aux tableaux si besoin).
- **Pas de symboles** `≥`, `<`, `×`. Écrire "supérieur ou égal à", "moins de", "fois", "par".
- **Garder uniquement** les symboles `%` (compact universel) et `$` (universel).
- Les chiffres en toutes lettres : `220 000 $` pas `220k$`, `5 mois` pas `5m`.
- **Mention sobre du Verdict** dans la prose pour chaque Top (intégré dans la phrase, pas en label).

## Structure du document

```markdown
# Priorisation par impact — [nom du projet] ([date])

[Préambule en prose, 1 paragraphe]

## Les cinq priorités

### 1. [Nom du use case]

Sponsor : [Prénom Nom] ([rôle ou département]).

[Paragraphe 5 à 8 lignes en prose narrative.]

### 2. [...]

[...]

### 3. [...]

### 4. [...]

### 5. [...]

## À chiffrer en atelier avant décision

[Use cases avec Verdict "À chiffrer prioritairement" ou "Stratégique
long-terme" pas dans le Top 5, en prose.]

## Dépendances et root causes

[Section OPTIONNELLE — incluse uniquement si la col 26 Dépend de a des
valeurs non vides OU si Step 2bis a consolidé des symptômes sous des root
causes. 2-3 paragraphes en prose.]

## Recommandations méthodologiques

[1 ou 2 paragraphes.]
```

## Préambule type

Le préambule en 1 paragraphe doit couvrir :

- Date d'analyse et nom du projet
- Sources d'inputs (transcripts, post-its, notes, inventaires)
- Nombre total de **candidats** identifiés en Step 1 et nombre de **root causes** retenues après Step 2bis
- Nombre retenu pour les cinq priorités
- Rappel discret de la règle d'or (chiffres durs en cellules, hypothèses en Notes)
- Rappel que l'effort technique n'est pas évalué dans cette note

Exemple :

```
Cette note synthétise l'analyse d'impact business des use cases identifiés
lors de l'atelier de découverte du 13 mai 2026 avec Maxime Soweif, Valérie
Audet-Nadon, Nicolas L et l'équipe Faspac. Vingt-deux candidats use cases
ont été extraits des transcripts, des post-its de cartographie des processus
et de la note d'analyse Ishikawa. Après application de la Step 2bis Root
Cause Analysis (consolidation de symptômes sous leurs causes profondes
actionnables), il reste douze root causes distinctes, dont cinq sont
retenues comme priorités dans cette note. Conformément à la règle d'or de
la méthode, seuls les chiffres durs cités par les sponsors entrent dans les
cellules Impact, les estimations indicatives sont reportées dans la colonne
Notes du CSV comme questions à poser en atelier de chiffrage. L'effort
technique de réalisation n'est pas évalué dans cette note et fera l'objet
d'une analyse complémentaire.
```

## Top entry type

Chaque entrée Top en 5 à 8 lignes de prose narrative. Pattern :

1. Une ligne pour Sponsor
2. Une à deux lignes pour la situation actuelle, avec les chiffres durs cités verbatim
3. Une à deux lignes pour l'impact documenté avec ventilation entre 2 ou 3 sources principales
4. Une ligne pour le verdict intégré dans la phrase
5. Une ligne pour les questions à chiffrer en atelier si pertinent (renvoi aux Notes du CSV)

Exemple avec mention root cause :

```
### 2. Agent d'estimation soumission from Business Central

Sponsor : Valérie Audet-Nadon (Finance et Commercial), avec sponsor exécutif
Maxime Lavoie (Direction Commerciale).

Le process de soumissions est aujourd'hui un goulot critique. Valérie est
la seule ressource sur la fonction estimation depuis le passage à Business
Central, et le cycle de génération d'une soumission est passé à trois
semaines là où la cible commerciale est de 24 heures. Cette root cause
consolide trois symptômes observés en atelier (saturation Valérie, lenteur
de cycle, erreurs marge) qui ont été identifiés en Step 2bis comme ayant
une seule automatisation possible. L'impact documenté chiffrable directement
(temps Valérie immobilisé) tourne autour de 90 000 $ par an, mais les deux
plus gros leviers (manque à gagner sur contrats retardés et erreurs marge
absorbées) ne sont pas encore chiffrés par le sponsor. Use case classé à
chiffrer prioritairement parce que la voix de Valérie et de la direction
est forte et qu'un chiffrage rapide en atelier pourrait débloquer plusieurs
centaines de milliers de dollars d'impact additionnel. Voir notes du CSV
pour les questions précises à poser au sponsor.
```

## Section "À chiffrer en atelier avant décision"

Pour chaque use case avec Verdict "À chiffrer prioritairement" ou "Stratégique long-terme" qui n'est pas dans le Top 5, format prose 3 à 5 lignes par use case mentionnant les questions clés à poser.

## Section "Dépendances et root causes" (optionnelle)

**Inclure uniquement si** :
- Au moins une ligne du CSV a une valeur non vide dans col 26 Dépend de, OU
- Step 2bis a consolidé plusieurs symptômes sous une root cause (le mentionner dans la synthèse aide le lecteur à comprendre pourquoi certains pain points apparents n'ont pas leur propre ligne)

Format : 2-3 paragraphes en prose, identifiant les chaînes critiques et les fondations à débloquer en premier.

Exemple (cas Faspac avec Data Lake comme fondation) :

```
## Dépendances et root causes

Trois des cinq priorités du Top partagent une dépendance commune au Data
Lake, qui apparaît dans la col 26 "Dépend de" des use cases Agent flagging
consommation matière première, Agent reconciliation Workestimity et
Dashboard recoster écarts. Le Data Lake lui-même n'apparaît pas dans le Top
5 par impact direct, car il n'adresse pas un pain point business mesurable
isolément. Sa construction est néanmoins préalable à la captation des
plusieurs centaines de milliers de dollars d'impact des trois use cases
consommateurs. L'ordre de lancement recommandé place donc le Data Lake en
mois un, suivi des trois use cases consommateurs en parallèle ou en
séquence selon les capacités équipe.

L'analyse root cause de Step 2bis a permis de regrouper plusieurs symptômes
sous des use cases actionnables, ce qui évite de triple-compter l'impact.
La saturation de Valérie sur les soumissions, par exemple, n'apparaît pas
comme un use case distinct mais comme un symptôme dont la root cause est
l'Agent d'estimation soumission from Business Central listé en priorité
deux. Cette consolidation rend la priorisation plus honnête mais demande
au lecteur de bien lire les pain points listés dans le CSV plutôt que de
chercher chaque symptôme observé en atelier comme une ligne séparée.
```

Si aucune dépendance et aucune consolidation Step 2bis : **omettre entièrement cette section**.

## Section "Recommandations méthodologiques"

1 ou 2 paragraphes qui rappellent les limites du chiffrage actuel, identifient les 2 ou 3 use cases qui méritent un atelier de chiffrage prioritaire, mentionnent la prochaine étape (futur skill effort).

Exemple :

```
L'impact total chiffré dans le CSV reste prudent car il s'appuie sur les
chiffres bruts partagés par les sponsors en atelier et sur des extrapolations
documentées (essentiellement heures par semaine fois coût horaire fully-
loaded). Pour les use cases en première priorité, un atelier de validation
avec les sponsors sera utile pour confirmer le manque à gagner commercial,
le taux d'erreur actuel et le coût exact des défauts. Les questions précises
sont listées dans la colonne Notes du CSV.

L'effort technique de réalisation (taille de build, coût agent à
l'exécution, dépendances techniques, intégrations) n'est pas évalué dans
cette note. Une analyse complémentaire d'effort viendra fermer la lecture
ROI complète. Le CSV de cette note de cadrage est conçu pour s'intégrer
directement avec ce futur skill complémentaire.
```

## Quality checks avant export

- [ ] Préambule mentionne date, sources, nombre de candidats avant et après Step 2bis, nombre de use cases retenus, règle d'or, scope impact-only
- [ ] Exactement 5 Top dans la section "Les cinq priorités"
- [ ] Chaque Top a sponsor + 5 à 8 lignes prose + verdict intégré
- [ ] Section "À chiffrer en atelier" présente avec use cases hors Top 5 ayant verdict À chiffrer ou Stratégique
- [ ] Section "Dépendances et root causes" présente si col 26 a des valeurs non vides ou si Step 2bis a consolidé des symptômes ; omise sinon
- [ ] Section "Recommandations méthodologiques" présente avec 1 ou 2 paragraphes
- [ ] Aucun em dash, aucun crochet, aucun pipe dans le corps du texte
- [ ] Chiffres en toutes lettres (220 000 $, pas 220k$)
- [ ] Document tient sur une page A4 imprimée (sauf si section Dépendances est nécessaire et l'étire à 1.5 page)
- [ ] Aucune section Méthodologie longue
