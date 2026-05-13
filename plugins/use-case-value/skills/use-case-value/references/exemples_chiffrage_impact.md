# Exemples de chiffrage Impact $ — frontière chiffre dur vs hypothèse

Ce document aide le LLM à décider, pour chaque colonne Impact $ (cols 13 à 18), ce qui peut entrer dans la cellule (chiffre dur ou formule simple sur chiffre dur) versus ce qui doit rester en Notes comme question à poser au sponsor.

## Principe général

**Va dans la cellule** :
- Valeur citée verbatim par un sponsor ou participant atelier
- Valeur tirée d'un tableau source (Excel, CSV, rapport)
- Formule simple appliquée à des chiffres durs (ex : heures par semaine citées fois 50 sem fois 80 $/h fully-loaded)

**Va en Notes (jamais dans la cellule)** :
- Estimation contextuelle du LLM ("je pense que…", "vraisemblablement…")
- Multiplicateur d'hypothèse appliqué à un chiffre dur (ex : "180 000 $ de non-conformité × 40 % captable") sauf si le 40 % vient explicitement du sponsor
- Extrapolation depuis un chiffre brut sans formule explicite (ex : "ils mentionnent ça donc ça doit valoir environ X")
- Toute valeur dont la dérivation n'est pas immédiatement traçable au texte

## Exemples par colonne

### Col 13 — Impact $ Temps Perdu

**Va dans la cellule** :

| Indice dans l'input | Valeur cellule | Justification |
|---------------------|----------------|---------------|
| "20 à 30 heures par semaine de matching" (Shirley) | `100 000 $` | Formule simple : 25 h/sem (milieu de fourchette) × 50 sem × 80 $/h |
| "Valérie passe 6 heures par soumission, fait 5 soumissions/sem" (Valérie) | `120 000 $` | Formule simple : 30 h/sem × 50 sem × 80 $/h |
| "Eric estime 10 h/sem sur la planif manuelle" (Eric) | `40 000 $` | Formule simple : 10 h/sem × 50 sem × 80 $/h |
| "L'équipe de 5 personnes y passe environ 2 h/jour chacune" (Nicolas) | `200 000 $` | Formule simple : 5 × 2 h × 250 jours × 80 $/h |

**Va en Notes** :

| Indice dans l'input | Cellule | Note |
|---------------------|---------|------|
| "On y passe beaucoup de temps" (sans chiffre) | `0` | `À chiffrer : combien d'heures par semaine en moyenne ? À titre indicatif, si 15 h/sem × 50 sem × 80 $/h = 60 000 $.` |
| "C'est énorme comme charge" (sans chiffre) | `0` | `À chiffrer : quel équivalent FTE actuel sur cette tâche ? Si 1 FTE = 100 000 $/an chargé.` |
| "Quelqu'un est dédié à temps plein" (sans précision sur le profil) | `0` | `À chiffrer : profil salarial du FTE dédié ? À 80 000 $ chargé moyen, +80 000 $ en Impact Temps Perdu.` |

### Col 14 — Impact $ Opportunités Manquées

**Va dans la cellule** (très restrictif, demande une citation explicite) :

| Indice dans l'input | Valeur cellule | Justification |
|---------------------|----------------|---------------|
| "On perd environ 10 contrats par an à 50 000 $ chacun à cause des délais" (Maxime) | `500 000 $` | Citation directe sponsor avec valeur ET volume |
| "Tableau commercial montre 8 leads perdus en 2025, valeur moyenne 75 000 $" | `600 000 $` | Tableau source explicite |
| "Notre churn moyen est 200 000 $ par an attribuable à la lenteur" (CEO) | `200 000 $` | Citation directe avec attribution explicite |

**Va en Notes** (cas typique) :

| Indice dans l'input | Cellule | Note |
|---------------------|---------|------|
| "Trois semaines au lieu de 24 heures, ça bloque" (Valérie, sans chiffre de contrat perdu) | `0` | `À chiffrer : combien de contrats sont perdus ou retardés par an à cause du délai 3 semaines ? À titre indicatif, si 5 contrats/an × 100 000 $ × 30 % probabilité capture, +150 000 $ en Impact Opportunités.` |
| "Les gros clients nous trouvent trop lents" (Direction) | `0` | `À chiffrer : combien d'appels d'offres gros clients perdus, quelle valeur typique ?` |
| "On rate des opportunités à cause de ça" (générique) | `0` | `À chiffrer : volume et valeur des opportunités ratées attribuables à ce process.` |

### Col 15 — Impact $ Coût Erreurs

**Va dans la cellule** :

| Indice dans l'input | Valeur cellule | Justification |
|---------------------|----------------|---------------|
| "180 000 $ de non-conformité par an" (Nicolas Direction Opérations) | `180 000 $` | Citation directe sponsor, valeur annuelle explicite |
| "On fait 2 erreurs majeures par an à 15 000 $ chacune en marge perdue" (Maxime) | `30 000 $` | Citation directe avec volume ET coût unitaire |
| "Le rework annuel coûte 50 000 $ documenté dans le rapport qualité 2025" | `50 000 $` | Source documentaire explicite |

**Va en Notes** :

| Indice dans l'input | Cellule | Note |
|---------------------|---------|------|
| "180 000 $ de non-conformité" + LLM veut appliquer "40 % captable" sans citation | `0` | `À chiffrer : quel pourcentage des 180 000 $ de non-conformité est captable par vision automatisée ? À titre indicatif, si 40 % captable, +72 000 $ en Impact Erreurs.` |
| "On fait des erreurs occasionnellement" (sans chiffre) | `0` | `À chiffrer : taux d'erreur actuel et coût moyen par erreur.` |
| "Les retours clients coûtent cher" (sans chiffre) | `0` | `À chiffrer : volume de retours annuels et coût moyen par retour.` |

**Cas frontière critique** : si le sponsor cite "180 000 $ de non-conformité" mais que le 40 % captable est une hypothèse LLM, alors la cellule reste à 0 et toute l'analyse va en Notes. Le 180 000 $ n'est pas l'impact captable de l'automatisation, c'est le pain actuel total.

### Col 16 — Impact $ Main d'Œuvre / Saturation

**Va dans la cellule** :

| Indice dans l'input | Valeur cellule | Justification |
|---------------------|----------------|---------------|
| "On va éviter d'embaucher 1 FTE chargé à 100 000 $" (engagement Direction) | `100 000 $` | Engagement explicite sponsor |
| "Heures sup annuelles de Shirley : 200 h à 60 $/h" (paie 2025) | `12 000 $` | Donnée source explicite |
| "Risque de burnout Valérie : remplacement coûterait 50 000 $ en formation" (DRH) | `50 000 $` | Citation DRH avec valeur explicite |

**Va en Notes** :

| Indice dans l'input | Cellule | Note |
|---------------------|---------|------|
| "Valérie est saturée" (sans plan d'embauche cité) | `0` | `À chiffrer : si Valérie part en burnout, coût de remplacement et perte de productivité estimés ?` |
| "On ne peut pas grossir sans embaucher" (Direction sans engagement chiffré) | `0` | `À chiffrer : combien de FTE à éviter à court terme si l'automatisation porte ses fruits ?` |
| "L'expert RH est sollicité trop souvent" (sans heures sup chiffrées) | `0` | `À chiffrer : heures supplémentaires de l'expert RH actuellement, à quel taux ?` |

### Col 17 — Impact $ Dépendances Externes

**Va dans la cellule** :

| Indice dans l'input | Valeur cellule | Justification |
|---------------------|----------------|---------------|
| "Pénalités contractuelles Sabre 30 000 $ par an documentées" (contrat) | `30 000 $` | Document source explicite |
| "Retards fournisseur causent 50 000 $ de stockage supplémentaire annuel" (Logistique) | `50 000 $` | Citation Logistique avec valeur explicite |

**Va en Notes** :

| Indice dans l'input | Cellule | Note |
|---------------------|---------|------|
| "Sabre est lent" (sans pénalité chiffrée) | `0` | `À chiffrer : coûts directs des retards Sabre (pénalités contractuelles, stockage, freight expédié).` |
| "On dépend de fournisseurs externes" (générique) | `0` | `À chiffrer : volume et coût des blocages dus aux dépendances externes par an.` |

### Col 18 — Impact $ Frictions Inter-Départementales

**Va dans la cellule** :

| Indice dans l'input | Valeur cellule | Justification |
|---------------------|----------------|---------------|
| "Estimation et Production échangent 5 emails par dossier, soit 30 min par dossier × 1500 dossiers/an × 70 $/h" (Maxime) | `52 500 $` | Citation avec volume ET temps unitaire |
| "Comptabilité refait 20 % du travail de Réception, soit 10 h/sem chacune" (Shirley) | `40 000 $` | Citation avec volume et temps |

**Va en Notes** :

| Indice dans l'input | Cellule | Note |
|---------------------|---------|------|
| "Il y a beaucoup d'allers-retours entre les équipes" (sans chiffre) | `0` | `À chiffrer : combien de communications inter-départementales par dossier, en moyenne temps consommé ?` |
| "On fait du double travail" (générique) | `0` | `À chiffrer : volume et durée des doublons actuels.` |

## Frontière critique : formule simple vs hypothèse LLM

**Test rapide** : "Si je dois inventer une variable de la formule (multiplicateur, pourcentage, probabilité, ratio) qui n'est pas citée par un sponsor ni issue d'un standard documenté comme le coût horaire fully-loaded par défaut, alors c'est une hypothèse LLM. Cellule = 0, note en Notes."

Exemples de variables qui transforment une "formule simple" en "hypothèse LLM" :

- **% captable** d'un pain actuel (40 % de 180 000 $ de non-conformité)
- **Probabilité de capture** d'opportunités (30 % de 5 contrats × 100 000 $)
- **Ratio de FTE évité** vs FTE total (0.5 FTE évité sur 1 FTE saturé)
- **Multiplicateur de gain** sur un coût existant (gain de 25 % sur les pénalités)

Si une variable de la formule vient de l'expérience MIA ou du LLM, l'ensemble de la formule va en Notes, pas dans la cellule. La cellule peut soit rester à 0, soit contenir une portion calculable sans hypothèse (ex : si "20 h/sem" est cité et que 80 $/h est le défaut documenté, la cellule peut contenir 20 × 50 × 80 = 80 000 $).

## Cas spécial : coût horaire fully-loaded par défaut

Le coût horaire de 80 $/h documenté dans `calculation_rules.md` est un **standard de la méthode**, pas une hypothèse LLM. Il peut être utilisé pour convertir des heures dures en Impact $ Temps Perdu sans que cela compte comme estimation LLM. À condition de documenter dans Notes : `Hypothèses : coût horaire fully-loaded 80 $/h utilisé.`

Si le sponsor cite un taux différent (ex : "nos estimateurs sont à 90 $/h chargé"), utiliser ce taux et documenter la source dans Notes.
