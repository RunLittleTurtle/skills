# Gabarit synthèse exécutive use-case-value

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

[Paragraphe de 5 à 8 lignes en prose narrative. Décrit la situation actuelle
avec les chiffres durs cités, mentionne le sponsor en action, énonce l'impact
documenté avec ventilation des sources principales, mentionne le verdict
intégré dans la phrase, ouvre vers les questions à chiffrer en atelier si
pertinent.]

### 2. [Nom du use case]

Sponsor : [Prénom Nom] ([rôle ou département]).

[Idem, 5 à 8 lignes en prose narrative.]

### 3. [Nom du use case]

[Idem]

### 4. [Nom du use case]

[Idem]

### 5. [Nom du use case]

[Idem]

## À chiffrer en atelier avant décision

[Use cases avec Verdict "À chiffrer prioritairement" ou "Stratégique
long-terme" qui ne sont pas déjà dans le Top 5. Format en prose, un
paragraphe par use case, mentionnant les questions clés à poser au sponsor.]

## Recommandations méthodologiques

[1 ou 2 paragraphes. Limites du chiffrage actuel, valeur d'un atelier de
validation avec les sponsors prioritaires, mention de la prochaine étape
d'évaluation d'effort technique qui sera traitée par un skill complémentaire.]
```

## Préambule type

Le préambule en 1 paragraphe doit couvrir :

- Date d'analyse et nom du projet
- Sources d'inputs (transcripts, post-its, notes, inventaires)
- Nombre total de use cases analysés
- Nombre retenu pour les cinq priorités
- Rappel discret de la règle d'or (chiffres durs en cellules, hypothèses en Notes)
- Rappel que l'effort technique n'est pas évalué dans cette note

Exemple :

```
Cette note synthétise l'analyse d'impact business des use cases identifiés
lors de l'atelier de découverte du 13 mai 2026 avec Maxime Soweif, Valérie
Audet-Nadon, Nicolas L et l'équipe Faspac. Quatorze use cases ont été passés
en revue à partir des transcripts de l'atelier, des post-its de cartographie
des processus et de la note d'analyse Ishikawa. La priorisation ci-dessous
retient les cinq use cases les plus prometteurs par impact business chiffré.
Conformément à la règle d'or de la méthode, seuls les chiffres durs cités
par les sponsors entrent dans les cellules Impact, les estimations
indicatives sont reportées dans la colonne Notes du CSV comme questions à
poser en atelier de chiffrage. L'effort technique de réalisation n'est pas
évalué dans cette note et fera l'objet d'une analyse complémentaire.
```

## Top entry type

Chaque entrée Top en 5 à 8 lignes de prose narrative. Pattern :

1. Une ligne pour Sponsor (`Sponsor : Prénom Nom (rôle ou département).`)
2. Une à deux lignes pour la situation actuelle, avec les chiffres durs cités verbatim si possible
3. Une à deux lignes pour l'impact documenté avec ventilation entre 2 ou 3 sources principales
4. Une ligne pour le verdict intégré dans la phrase ("Use case classé en première priorité parce que…")
5. Une ligne pour les questions à chiffrer en atelier si pertinent (renvoi aux Notes du CSV)

Exemple Top 1 (basé sur les inputs Faspac avec règle d'or appliquée) :

```
### 1. Matching PO / Facture / Réception

Sponsor : Shirley (Comptabilité), avec relais Nicolas L à la Direction.

Le rapprochement manuel entre bon de commande, facture et réception mobilise
20 à 30 heures par semaine côté Shirley, chiffre validé par le sponsor en
atelier. Sur cette seule base, l'impact annuel documenté est d'environ
100 000 $ de temps comptabilité absorbé sur du repetitive matching à coût
horaire fully-loaded. Le coût des erreurs de réconciliation et l'impact sur
les retards de paiement restent à chiffrer en atelier dédié. Disponibilité
des données élevée (Business Central plus factures scannées). Quick win
prioritaire compte tenu de la combinaison impact documenté solide et
disponibilité des données.
```

Exemple Top 2 avec verdict "À chiffrer prioritairement" :

```
### 2. Soumissions clients

Sponsor : Valérie Audet-Nadon (Finance et Commercial).

Le process de soumissions est aujourd'hui un goulot critique. Valérie est
la seule ressource sur la fonction estimation depuis le passage à Business
Central, et le cycle de génération d'une soumission est passé à trois
semaines là où la cible commerciale est de 24 heures. La direction a
confirmé en atelier que cette lenteur bloque les gros appels d'offres
(Agropeur, Olymel). L'impact documenté chiffrable directement (temps Valérie
immobilisé) tourne autour de 90 000 $ par an, mais les deux plus gros
leviers (manque à gagner sur contrats retardés et erreurs d'estimation
absorbées en marge) ne sont pas encore chiffrés par le sponsor. Use case
classé à chiffrer prioritairement parce que la voix de Valérie et de la
direction est forte et qu'un chiffrage rapide en atelier pourrait débloquer
plusieurs centaines de milliers de dollars d'impact additionnel. Voir notes
du CSV pour les questions précises à poser au sponsor.
```

## Section "À chiffrer en atelier avant décision"

Pour chaque use case avec Verdict "À chiffrer prioritairement" ou "Stratégique long-terme" qui n'est pas dans le Top 5 :

```
Estimation prévisionnelle de chaîne par Eric (Production). Le pain point
est réel et a été nommé en atelier par Eric et Maxime, mais aucun volume
ni coût n'est documenté dans les inputs actuels. Un atelier chiffrage
dédié serait nécessaire avant cadrage : combien d'écart annuel entre
estimé et réel chaîne, combien de temps de planification absorbé,
combien de pertes matière première liées à la mauvaise prévision.
```

Format en prose, 3 à 5 lignes par use case.

## Section "Recommandations méthodologiques"

1 ou 2 paragraphes qui :

- Rappellent que l'impact documenté est prudent (chiffres durs uniquement)
- Identifient les 2 ou 3 use cases qui méritent un atelier de chiffrage prioritaire avec leurs sponsors
- Mentionnent la prochaine étape (analyse d'effort technique via un skill complémentaire)
- Suggèrent un ordre d'investigation si pertinent

Exemple :

```
L'impact total chiffré dans le CSV reste prudent : il s'appuie sur les
chiffres bruts partagés par les sponsors en atelier et sur des extrapolations
documentées (essentiellement heures par semaine fois coût horaire fully-
loaded). Pour les use cases en première priorité (Matching PO, Soumissions
clients, Contrôle qualité visuel), un atelier de validation avec les
sponsors sera utile pour confirmer le manque à gagner commercial, le taux
d'erreur actuel et le coût exact des défauts. Les questions précises sont
listées dans la colonne Notes du CSV.

L'effort technique de réalisation (taille de build, coût agent à l'exécution,
dépendances techniques, intégrations) n'est pas évalué dans cette note. Une
analyse complémentaire d'effort viendra fermer la lecture ROI complète. Le
CSV de cette note de cadrage est conçu pour s'intégrer directement avec ce
futur skill complémentaire.
```

## Quality checks avant export

- [ ] Préambule mentionne date, sources, nombre de use cases, règle d'or, scope impact-only
- [ ] Exactement 5 Top dans la section "Les cinq priorités"
- [ ] Chaque Top a sponsor + 5 à 8 lignes prose + verdict intégré
- [ ] Section "À chiffrer en atelier" présente avec use cases hors Top 5 ayant verdict À chiffrer ou Stratégique
- [ ] Section "Recommandations méthodologiques" présente avec 1 ou 2 paragraphes
- [ ] Aucun em dash, aucun crochet, aucun pipe dans le corps du texte
- [ ] Chiffres en toutes lettres (220 000 $, pas 220k$)
- [ ] Document tient sur une page A4 imprimée
- [ ] Aucune section Méthodologie longue, aucune Lectures transversales séparée
