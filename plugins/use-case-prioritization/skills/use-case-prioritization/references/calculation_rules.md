# Règles de calcul et valeurs par défaut

Ce document définit les formules exactes, les seuils, et les valeurs par défaut utilisées pour générer le CSV de priorisation. À lire après `framework_v4.md` pour les détails de calcul.

## Barème T-shirt (col 28 Build Base)

Si l'utilisateur fournit seulement la taille T-shirt sans montant, utiliser le lookup suivant:

| T-shirt | Build Base ($) | Effort indicatif |
|---|---|---|
| XS | 10 000 | < 2 semaines |
| S | 25 000 | 3-4 semaines |
| M | 50 000 | 6-8 semaines |
| L | 90 000 | 10-12 semaines |
| XL | 150 000 | 14+ semaines |

Si l'architecte fournit un montant override (ex: "M mais plutôt 65k"), utiliser le montant override.

## Valeurs par défaut

**Taux Réalisation (col 25)**: 0.6 par défaut si non spécifié. Refléter le change management overhead (Prosci/McKinsey practice).
- 0.5 pour use cases à fort impact organisationnel (adoption complexe)
- 0.6 pour cas standards
- 0.7 pour automatisations à faible friction utilisateur

**Risque (col 34)**: 3 par défaut si non spécifié.
- 1-2: Risque faible (technique connue, sponsor fort, données simples)
- 3: Risque moyen
- 4-5: Risque élevé (technique inconnue, pas de sponsor, données problématiques)

**Run Annuel (col 30)**: Build Base × 0.15 si vide.
- Heuristique 15% du TCO IT classique
- Pour AI haute volumétrie, cette valeur sous-estime. Documenter dans Notes Architecte.

## Formules de calcul

### Suitability Score (col 14)
```
Suitability = MOYENNE(col 6 à col 13) × 20
```
Donne un score sur 100.

### Gain Heures Annuel (col 23)
```
Gain_H = Volume_Qte × (Temps_Actuel - Temps_Cible) × Personnes_Impactees
```
Si Temps_Actuel = 0 (cas pertes évitées directes uniquement), Gain_H = 0.

### Bénéfice Brut Annuel (col 24)
```
SI Temps_Actuel = 0:
  Benefice_Brut = Pertes_Evitees_Directes
SINON:
  Benefice_Brut = Gain_H × Cout_Horaire + Pertes_Evitees_Directes (si applicable)
```

### Bénéfice Net Annuel (col 26)
```
Benefice_Net = Benefice_Brut × Taux_Realisation
```

### Build Base (col 28)
```
SI montant fourni: utiliser le montant
SINON: lookup table t-shirt (voir barème ci-dessus)
SI ni montant ni t-shirt: laisser vide (critique pour confiance)
```

### Run Annuel (col 30)
```
SI rempli par architecte: utiliser la valeur
SINON: Run = Build_Base × 0.15
```

### Coût An 1 Total (col 31)
```
Cout_An1 = Build_Base + Run_Annuel
```

### Payback en mois (col 32)
```
SI (Benefice_Net - Run_Annuel) <= 0:
  Payback = 999 (jamais rentable)
SINON:
  Payback = Build_Base × 12 / (Benefice_Net - Run_Annuel)
```

### ROI An 1 en % (col 33)
```
ROI_An1 = (Benefice_Net - Cout_An1) / Cout_An1 × 100
```
Note: souvent négatif en An 1 même pour bons projets.

### Score Priorité (col 35)
```
Score = Benefice_Net / (Build_Base × Risque)
```
Plus élevé = meilleur.

## Calcul de la confiance

### Liste des 20 colonnes critiques (pour col 36 Nb Colonnes Critiques Remplies)

1. Pain Point (col 2)
2. Sponsor / Owner (col 4) - "À identifier" compte comme NON rempli
3. Département (col 5)
4. Volume Standardisation (col 6)
5. Règles Claires (col 7)
6. Disponibilité Données (col 8)
7. Stabilité Processus (col 9)
8. Faible Taux Exceptions (col 10)
9. Impact Erreurs Actuelles (col 11)
10. Alignement Stratégique (col 12)
11. Sponsor Engagé (col 13)
12. Nb Intégrations Identifiées (col 15)
13. Volume Qté (col 17)
14. Temps Actuel (col 18)
15. Temps Cible (col 19)
16. Personnes Impactées (col 20)
17. Coût Horaire OU Pertes Évitées Directes (cols 21 ou 22, une des deux suffit)
18. Taux Réalisation (col 25)
19. Taille Build (col 27)
20. Risque (col 34)

**Règles de comptage**:
- Valeur numérique = compté comme rempli
- "N/A" (string) = compté comme rempli (la question a été considérée)
- Vide = NON compté
- "À identifier" / "À valider" pour Sponsor = NON compté
- 0 = compté comme rempli (zéro est une valeur valide)

### Confiance Données en % (col 39)
```
Conf_Donnees = Nb_Colonnes_Critiques_Remplies / 20 × 100
```

### Qualité Source (col 37) - input manuel 1-5
- 5: Transcript validé + chiffres confirmés par client
- 4: Transcript ou document client
- 3: Note d'atelier validée
- 2: Note d'atelier non validée
- 1: Estimation analyste / hypothèse

Si non spécifié par l'utilisateur, estimer à partir du type d'input:
- Image post-its + transcript: 3
- Photo post-its seule: 2
- Table de processus structurée: 4
- Transcript de meeting seul: 3
- Notes manuelles: 2

### Cohérence Check (col 38) - input manuel 1-5
Vérifier les cohérences classiques:
- Volume × Temps × Personnes correspond-il à un effort plausible?
- Pertes évitées sont-elles cohérentes avec l'impact erreurs?
- Sponsor engagé et Alignement stratégique sont-ils cohérents?

Defaults:
- 5: Tout colle, valeurs s'expliquent
- 4: Une légère incohérence acceptable
- 3: Une incohérence modérée
- 2: Plusieurs incohérences
- 1: Valeurs suspectes ou contradictoires

Si pas d'info pour évaluer, défaut = 3.

### Confiance Globale en % (col 40)
```
Conf_Globale = 0.4 × Conf_Donnees + 0.35 × (Qualite_Source × 20) + 0.25 × (Coherence × 20)
```

## Seuils de verdicts

### Verdict Confiance (col 41)
| Confiance Globale | Verdict |
|---|---|
| ≥ 80% | Fiable |
| 60-79% | A valider 1-2 points |
| 40-59% | Hypotheses fortes |
| < 40% | Atelier de validation requis |

### Verdict ROI (col 42)
Basé sur Payback en mois:
| Payback | Verdict |
|---|---|
| < 6 mois | Quick win |
| 6-12 mois | Go |
| 12-18 mois | A challenger |
| 18-24 mois | Strategique seulement |
| > 24 mois (incluant 999) | Pass |

## Conventions de remplissage

### Quand le Pain Point n'est pas explicite
Reformuler le problème central en une phrase. Exemples:
- Post-it "30h/semaine" + "erreurs scrap" → "30h/sem erreurs production, scrap milliers de $"
- Transcript "on perd beaucoup d'argent en non-conformité" + chiffre 160k$ → "160k$ pertes non-conformité par an"

### Quand le Sponsor est nommé
Si le client cite une personne (ex: "Robert au VP ventes s'occupe de ça"), mettre "Robert VPventes" comme Sponsor / Owner.
Si seulement un département est mentionné, mettre le département dans Département mais "A identifier" dans Sponsor.

### Quand les volumes sont approximatifs
Si le client dit "environ 12k-15k par année", utiliser la médiane (13500) et noter dans Notes "Volume entre 12k et 15k confirmé".

### Quand le Temps Cible n'est pas spécifié
Heuristique: viser 20% du Temps Actuel pour les processus AI-assistés. Documenter dans Notes "Temps cible estimé à 20% (à valider)".

### Quand l'input mentionne des montants en $
- Si lié à des pertes ou erreurs: aller dans Pertes Évitées Directes ($) (col 22)
- Si lié à un coût de personne: aller dans Coût Horaire ($) (col 21)
- Si lié à un build: aller dans Build Base ($) (col 28)
