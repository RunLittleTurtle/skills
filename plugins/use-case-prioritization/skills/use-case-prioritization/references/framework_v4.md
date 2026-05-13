# Framework de priorisation des use cases AI (v4)

Référence des 43 colonnes du CSV v4. Combine triage qualitatif (Suitability), business case quantitatif (ROI), output architecte minimal, et score de confiance composite.

## Principe directeur v4

**Séparation claire des rôles** :
- **Analyste fonctionnel (toi)** : remplit cols 1-26 et 30-43. Cartographie + chiffrage business avec le client.
- **Architecte dev** : remplit cols 27-29. Output minimal: T-shirt size + montant + note de raisonnement.

L'architecte fait son calcul mental (intégrations, complexité, données, etc.) et synthétise en une seule taille T-shirt. Le détail technique reste dans sa tête, sa traçabilité va dans "Notes Architecte".

## Conventions transversales

- **(input)** : à remplir manuellement
- **Numérique** : chiffre direct ou échelle 1-5
- **N/A** : la question ne s'applique pas, compte comme rempli pour la confiance
- **Vide** : pas encore évalué, fait baisser la confiance
- **[ARCHI]** : à remplir par l'architecte dev, pas l'analyste

---

## Bloc 1: Identité (cols 1-5)

### 1. Use Case
**Rôle**: Nom court du processus candidat à automatiser, idéalement formulé comme un verbe d'action.
**Source**: BABOK 10.10 Business Case, convention agence.

### 2. Pain Point
**Rôle**: Description du problème actuel, citation directe du client si possible.
**Source**: BABOK 10.10, atelier qualitatif (SVA / cartographie).

### 3. Source Atelier
**Rôle**: Indique d'où vient l'information (couleur des post-its, transcript, document client). Permet de tracer la provenance.
**Source**: Convention interne pour traçabilité.

### 4. Sponsor / Owner
**Rôle**: Personne nommée qui porte le processus côté client. "À identifier" = drapeau rouge, sans sponsor le projet échoue.
**Source**: BABOK 10.18 Stakeholder List, Prosci ADKAR.

### 5. Département
**Rôle**: Unité organisationnelle qui possède le processus. Sert au regroupement et à l'analyse de portefeuille.
**Source**: Gouvernance organisationnelle.

---

## Bloc 2: Suitability Score qualitatif (cols 6-14)

Filtre amont rapide inspiré de UiPath/Deloitte. Chaque critère noté 1-5, score composite sur 100.

### 6. Volume / Standardisation (1-5)
**Rôle**: Le processus est-il assez fréquent et standardisé pour valoir l'automatisation? 1 = rare et variable, 5 = haut volume très standard.
**Source**: UiPath Process Assessment Framework.

### 7. Règles Claires (1-5)
**Rôle**: Les règles métier sont-elles documentées et déterministes? 1 = jugement humain partout, 5 = règles écrites et stables.
**Source**: Deloitte Automation Suitability.

### 8. Disponibilité Données (1-5)
**Rôle**: Les données nécessaires existent-elles et sont-elles accessibles? 1 = papier/manuel/inexistantes, 5 = APIs propres avec données prêtes. *Renommée v4 (anciennement "Données Structurées Disponibles").*
**Source**: Gartner Data Readiness Assessment.

### 9. Stabilité Processus (1-5)
**Rôle**: Le processus change-t-il souvent? 1 = change tous les mois, 5 = stable depuis des années.
**Source**: UiPath - critère stabilité (risque d'obsolescence de l'automatisation).

### 10. Faible Taux Exceptions (1-5)
**Rôle**: Combien de cas hors-norme à coder spécialement? 1 = 50%+ d'exceptions, 5 = moins de 5%.
**Source**: Lean Six Sigma - variabilité de processus.

### 11. Impact Erreurs Actuelles (1-5)
**Rôle**: Quelle est la conséquence des erreurs humaines actuelles? 1 = cosmétique, 5 = pertes financières ou risque réglementaire.
**Source**: Lean - Cost of Poor Quality (Juran).

### 12. Alignement Stratégique (1-5)
**Rôle**: Le use case s'aligne-t-il avec les OKR ou la vision du client? 1 = tangentiel, 5 = directement lié à un objectif stratégique.
**Source**: SAFe Strategic Themes, framework OKR.

### 13. Sponsor Engagé (1-5)
**Rôle**: Niveau d'engagement du sponsor identifié. 1 = pas de sponsor, 5 = sponsor actif qui débloque ressources.
**Source**: Prosci ADKAR - dimension Sponsorship.

### 14. Suitability Score (0-100)
**Rôle**: Score composite des 8 critères ci-dessus. Permet de filtrer rapidement avant de chiffrer.
**Source**: UiPath Suitability composite. Seuil pratique: en-dessous de 50, passer; au-dessus de 70, chiffrer.

---

## Bloc 3: Cartographie SI fonctionnelle (col 15)

### 15. Nb Intégrations Identifiées
**Rôle**: Information fonctionnelle captée en atelier client (ex: "c'est connecté à Clio et Outlook"). N'alimente aucune formule directement, mais informe l'architecte dans son calcul mental de T-shirt. *Nouvelle v4.*
**Source**: Cartographie SI - info fonctionnelle atelier.

---

## Bloc 4: Quantitatif économique (cols 16-26)

### 16. Volume Annuel (unité)
**Rôle**: Unité de fréquence (jour, semaine, mois, année). Précise comment interpréter le Volume Qté.
**Source**: Convention agence.

### 17. Volume Qté
**Rôle**: Nombre d'occurrences du processus par an, dans l'unité choisie.
**Source**: Ingénierie industrielle (Taylor, Gilbreth).

### 18. Temps Actuel (h/occ)
**Rôle**: Heures consommées par occurrence du processus aujourd'hui.
**Source**: Time and Motion Study (MTM, MOST).

### 19. Temps Cible (h/occ)
**Rôle**: Heures consommées par occurrence après automatisation. Souvent 10-30% du temps actuel pour les processus AI-assistés.
**Source**: Pratique RPA / Hyperautomation (UiPath, Forrester).

### 20. Personnes Impactées
**Rôle**: Nombre de personnes qui exécutent ce processus. Multiplie le gain horaire.
**Source**: Activity Based Costing (Kaplan, Cooper).

### 21. Coût Horaire ($)
**Rôle**: Coût horaire chargé (salaire + charges + overhead) de la personne moyenne qui fait ce processus.
**Source**: Comptabilité analytique - Fully Loaded Cost (Horngren, Kaplan HBS).

### 22. Pertes Évitées Directes ($)
**Rôle**: Montant annuel de pertes financières évitées (non-conformité, scrap, fraude). Distinct du temps économisé. Utiliser quand le bénéfice n'est pas du temps mais de la perte directe.
**Source**: Lean - Cost of Poor Quality (Juran).

### 23. Gain Heures Annuel
**Rôle**: Heures économisées par an. Formule: Vol × (T_actuel - T_cible) × Personnes.
**Source**: Time Study × Volume - ingénierie industrielle.

### 24. Bénéfice Brut Annuel ($)
**Rôle**: Valeur monétaire théorique du gain. Formule: Gain Heures × Coût Horaire + Pertes Évitées.
**Source**: Productivité × Managerial Accounting.

### 25. Taux Réalisation (0-1)
**Rôle**: Haircut sur le bénéfice brut. Reflète qu'on ne récupère jamais 100% du temps gagné. 0.5-0.7 typique.
**Source**: Change Management - Benefit Realization Factor (Prosci, pratique McKinsey/BCG).

### 26. Bénéfice Net Annuel ($)
**Rôle**: Bénéfice réaliste après haircut. C'est le chiffre utilisé pour Payback, ROI, Score Priorité. Formule: Brut × Taux.
**Source**: Pratique conseil transformation.

---

## Bloc 5: Output architecte (cols 27-29)

C'est ici que l'architecte synthétise sa réflexion technique. Il utilise les inputs fonctionnels (cols 8, 15) comme guide, mais ne remplit que 3 colonnes.

### 27. Taille Build (T-shirt) [ARCHI]
**Rôle**: Estimation de l'effort de développement. XS/S/M/L/XL. Output principal de l'architecte après son calcul mental.
**Source**: Agile Estimation (Mike Cohn 2005), Scrum/SAFe.

### 28. Build Base ($) [ARCHI]
**Rôle**: Montant en dollars. Par défaut lookup du barème agence MIA. L'architecte peut overrider si la complexité réelle s'écarte du barème.
**Source**: Barème agence MIA (XS=10k, S=25k, M=50k, L=90k, XL=150k). Calibré par expérience projets passés.

### 29. Notes Architecte
**Rôle**: Raisonnement libre de l'architecte. Pourquoi M et pas L? Quelles intégrations? Risques techniques? Sert à la traçabilité et à la révision future. *Nouvelle v4.*
**Source**: Traçabilité du raisonnement architecte.

---

## Bloc 6: Coûts d'opération et ROI (cols 30-35)

### 30. Run Annuel ($)
**Rôle**: Coût annuel de fonctionnement (LLM, infra, monitoring, maintenance). Optionnel: si vide, formule par défaut Build × 15%.

**ATTENTION HEURISTIQUE AI**: Le 15% vient du TCO classique IT (Gartner). Pour l'AI, le run varie beaucoup selon le volume d'appels et le modèle:
- Faible volume (< 1k appels/mois): 5-10% du build
- Moyen (1k-50k/mois): 10-20%
- Élevé (50k-500k/mois): 20-40%
- Très élevé (500k+/mois): peut dépasser le build

Si l'architecte a fait une vraie estimation tokens, il met le vrai chiffre. Sinon, 15% est une heuristique conservatrice pour usage moyen.

**Source**: TCO IT - Gartner / TBM Council, ajusté pour AI.

### 31. Coût An 1 Total ($)
**Rôle**: Investissement total la première année. Formule: Build Base + Run Annuel.
**Source**: Comptabilité analytique.

### 32. Payback (mois)
**Rôle**: Nombre de mois pour récupérer l'investissement. Formule: Build Base × 12 / (Bénéfice Net - Run Annuel). Métrique la plus parlante pour stakeholders.
**Source**: Finance corporative - Payback Period (Fisher 1930, Brealey-Myers).

### 33. ROI An 1 (%)
**Rôle**: Rendement la première année. Formule: (Bénéfice Net - Coût An 1) / Coût An 1. Souvent négatif An 1 même pour bons projets.
**Source**: Finance corporative - ROI standard.

### 34. Risque (1-5)
**Rôle**: Risque global d'échec du projet (adoption, technique, organisationnel). 1 = faible, 5 = très élevé.
**Source**: BABOK 10.38 Risk Analysis.

### 35. Score Priorité
**Rôle**: Score composite pour ranker les use cases. Formule: Bénéfice Net / (Build Base × Risque). Plus élevé = meilleur.
**Source**: WSJF simplifié (SAFe).

---

## Bloc 7: Méta-confiance (cols 36-41)

### 36. Nb Colonnes Critiques Remplies
**Rôle**: Compte sur les 20 colonnes considérées critiques. N/A compte comme rempli, vide ne compte pas.
**Source**: Data Quality Management - Completeness (DAMA-DMBOK).

### 37. Qualité Source (1-5)
**Rôle**: Fiabilité de la source des données. 5 = transcript validé client, 3 = note d'atelier validée, 1 = estimation analyste.
**Source**: Triangulation méthodologique.

### 38. Cohérence Check (1-5)
**Rôle**: Les valeurs sont-elles cohérentes entre elles? 5 = tout colle, 1 = incohérence majeure.
**Source**: DAMA-DMBOK Data Quality - Consistency.

### 39. Confiance Données (%)
**Rôle**: Pourcentage de complétude. Formule: Nb_Rempli / 20 × 100.
**Source**: Data Completeness Ratio (DAMA).

### 40. Confiance Globale (%)
**Rôle**: Score composite final de fiabilité. Formule: 0.4 × Confiance Données + 0.35 × Qualité Source × 20 + 0.25 × Cohérence × 20.
**Source**: Composite confidence score, pratique data science / BI.

### 41. Verdict Confiance
**Rôle**: Lecture rapide du score. Seuils: ≥80% Fiable, 60-80% À valider 1-2 points, 40-60% Hypothèses fortes, <40% Atelier requis.
**Source**: Seuils de confiance, pratique BI/Analytics.

---

## Bloc 8: Décision (cols 42-43)

### 42. Verdict ROI
**Rôle**: Lecture rapide de la viabilité économique. Seuils Payback: <6m Quick win, 6-12m Go, 12-18m À challenger, 18-24m Stratégique seulement, >24m Pass.
**Source**: Decision Rules - pratique cabinet conseil et benchmarks RPA.

### 43. Notes
**Rôle**: Commentaires libres analyste, drapeaux rouges, prochaines étapes, hypothèses à valider.
**Source**: BABOK 10.10 - section Assumptions and Risks.

---

## Liste des 20 colonnes critiques pour la confiance

1. Pain Point (col 2)
2. Sponsor / Owner (col 4)
3. Département (col 5)
4-11. Les 8 critères Suitability (cols 6-13)
12. Nb Intégrations Identifiées (col 15)
13. Volume Qté (col 17)
14. Temps Actuel (col 18)
15. Temps Cible (col 19)
16. Personnes Impactées (col 20)
17. Coût Horaire OU Pertes Évitées (cols 21-22, une des deux)
18. Taux Réalisation (col 25)
19. Taille Build (col 27)
20. Risque (col 34)

Note: Build Base ($), Notes Architecte et Run Annuel ne sont pas critiques. Le T-shirt suffit comme input architecte minimal.

---

## Workflow d'utilisation

**Étape 1 (atelier client, analyste)**: Remplir cols 1-15. Pain points, sponsor, 8 critères Suitability, intégrations identifiées.

**Étape 2 (analyste, post-atelier)**: Si Suitability < 50, marquer en Pass et passer. Sinon, remplir cols 16-26 (quantitatif économique).

**Étape 3 (architecte)**: Lire la fiche, faire son calcul mental, livrer cols 27-29 (T-shirt, Build Base optionnel, Notes). Le Run reste vide sauf si estimation tokens disponible.

**Étape 4 (analyste)**: Remplir Risque (col 34). Tous les calculs ROI se mettent à jour.

**Étape 5 (analyste)**: Évaluer Qualité Source et Cohérence (cols 37-38). Confiance Globale se calcule.

**Étape 6 (décision)**: Trier par Score Priorité. Filtrer par Verdict ROI = Quick win ou Go. Confirmer Verdict Confiance ≥ "À valider 1-2 points".

**Étape 7**: Pour les use cases retenus, passer en scoping détaillé (SVA mapping, PRD, etc.).

---

## Changelog v3 → v4

**Retiré** :
- Complexité Données [ARCHI] (la question était déjà couverte par Disponibilité Données)
- Nb Intégrations [ARCHI] (devenu fonctionnel, info atelier)
- Build Ajusté ($) (redondant avec T-shirt)
- Formule de Build Ajusté basée sur Complexité × 0.05 + Intégrations × 0.05

**Renommé** :
- "Données Structurées Disponibles" → "Disponibilité Données" (plus large)

**Ajouté** :
- Nb Intégrations Identifiées (col 15, fonctionnel)
- Notes Architecte (col 29, traçabilité)
- Section explicite sur la variabilité du Run AI

**Confiance recalibrée** :
- 21 colonnes critiques → 20 colonnes critiques
- Nb Intégrations Identifiées (fonctionnel) reste critique
- Build Base, Notes Architecte, Run Annuel ne sont pas critiques (Build Base auto-rempli, Notes optionnelle, Run rarement connu)
