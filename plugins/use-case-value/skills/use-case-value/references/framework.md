# Framework use-case-value — 26 colonnes, focus impact chiffré

Ce framework adapte quatre sources méthodologiques à un usage strictement orienté **valeur business chiffrable** :

- **BABOK 10.10 Business Case** (IIBA Business Analysis Body of Knowledge v3) : pour la décomposition de la valeur en sources distinctes (financières directes, opportunités, risques évités, valeur intangible).
- **SAFe WSJF (Weighted Shortest Job First)** : adapté en mode impact-only. On retient la dimension Cost of Delay (urgence × valeur) mais on retire la Job Size (effort), qui sera traitée par un futur skill séparé.
- **5 Whys (Taiichi Ohno, Toyota Production System) + Ishikawa Diagram (Kaoru Ishikawa)** : pour la Step 2bis d'analyse root cause. Permet d'identifier le niveau actionnable d'automatisation (la cause profonde captable par un agent) vs les symptômes visibles.
- **DAMA-DMBOK Data Quality Dimensions** (Data Management Association International) : pour le critère Disponibilité des données (col 20).

Le skill se distingue de `use-case-prioritization` v2.2 par quatre choix structurants :

1. **Aucune dimension d'effort** (pas de Build, Run, Payback, ROI An 1). Permet de comparer les use cases sur leur valeur intrinsèque, indépendamment du coût technique de réalisation.

2. **Six sources d'Impact $ explicites** (vs un seul Bénéfice Net dans v2.2). Capture la valeur qui vient des opportunités commerciales, des erreurs absorbées en marge, de la saturation de ressources critiques, des dépendances externes et des frictions inter-départementales, pas uniquement du temps directement libéré.

3. **Règle d'or stricte sur les estimations** : les cellules Impact $ ne contiennent que des chiffres durs cités ou des formules simples sur chiffres durs ; les hypothèses vont en Notes comme questions à poser au sponsor. Empêche le LLM de gonfler artificiellement le score de priorisation.

4. **Step 2bis Root Cause + colonne Dépend de** : avant chiffrage, le LLM applique un mini 5 Whys ou un Ishikawa léger pour identifier si chaque candidat use case est une root cause actionnable (à enregistrer) ou un symptôme d'un autre process (à fusionner dans la root cause). La colonne Dépend de capture les dépendances inter-use-cases pour la lecture transversale dans la synthèse.

---

## Liste des 26 colonnes

### Bloc 1 — Identification (cols 1 à 5)

| # | Colonne | Type | Source |
|---|---------|------|--------|
| 1 | Use Case | texte | input |
| 2 | Description | texte 1-2 phrases | input ou LLM synthétise |
| 3 | Sponsor | texte | input |
| 4 | Département principal | texte | input |
| 5 | Type Agent | texte court | LLM catégorise |

**Type Agent (col 5)** — valeurs autorisées :
- `email auto` : tri, classification, réponse automatisée de courriels
- `OCR` : extraction de documents (factures, BL, formulaires, contrats)
- `dashboard genAI` : génération de rapports ou dashboards dynamiques par LLM
- `RAG` : chatbot interne sur base documentaire (procédures, FAQ, manuels)
- `vision` : détection de défauts ou contrôle qualité par vision
- `RPA` : automatisation processuelle robotique sur SI legacy
- `autre` : tout autre type (préciser dans Notes)

### Bloc 2 — Contexte d'usage (cols 6 à 9)

| # | Colonne | Type | Rubrique |
|---|---------|------|----------|
| 6 | Volume Annuel Qté | nombre | nombre de runs, instances ou unités traitées par an |
| 7 | Volume Annuel Unité | texte court | factures/an, soumissions/an, courriels/an, défauts/jour, etc. |
| 8 | Personnes Affectées (Nb) | nombre | combien dans l'organisation sont touchées au quotidien par le pain point |
| 9 | Transversalité | 1-3 + label | voir rubrique ci-dessous |

**Transversalité (col 9)** — rubrique :
- `1 - un département` : un seul département ou équipe est impacté
- `2 - inter-départemental` : deux ou trois départements collaborent ou sont touchés
- `3 - toute l'organisation` : impact transverse, touche l'ensemble de l'organigramme

### Bloc 3 — Pain points (cols 10 à 12)

| # | Colonne | Type | Source |
|---|---------|------|--------|
| 10 | Pain Point Principal | texte 1 phrase, obligatoire | input |
| 11 | Pain Point #2 | texte 1 phrase, optionnel | input |
| 12 | Pain Point #3 | texte 1 phrase, optionnel | input |

Format suggéré : `[Acteur] [verbe d'action] [contrainte ou perte]`. Exemple : `Valérie passe 6 heures par soumission alors que la cible commerciale est 24 heures end-to-end`.

**Note Step 2bis** : si plusieurs symptômes apparents ont une root cause commune, ils sont consolidés dans Pain Points #1 à #3 d'une **seule ligne** (la root cause actionnable). Ne pas créer une ligne par symptôme.

### Bloc 4 — Sources d'Impact $ annuel (cols 13 à 18)

**Règle d'or absolue** : ces 6 colonnes acceptent uniquement des chiffres durs cités ou validés par le sponsor / participant atelier / table source, ou des formules simples sur ces chiffres durs. Tout le reste va en Notes (col 25) sous forme de question structurée.

| # | Colonne | Définition | Exemple de chiffrage direct |
|---|---------|------------|-----------------------------|
| 13 | Impact $ Temps Perdu | Coût annuel du temps humain absorbé par les tâches répétitives du process actuel | `25 h/sem × 50 sem × 80 $/h = 100 000 $` (heures citées par sponsor) |
| 14 | Impact $ Opportunités Manquées | Revenu retardé ou perdu chiffré par le sponsor : contrats lents, leads perdus, fenêtres commerciales ratées | `10 contrats/an perdus × 50 000 $ valeur moyenne = 500 000 $` (chiffre cité par sponsor) |
| 15 | Impact $ Coût Erreurs | Coût annuel des erreurs humaines actuelles (rework, retours produits, marges absorbées) | `180 000 $ de non-conformité annuelle citée par la Direction` |
| 16 | Impact $ Main d'Œuvre / Saturation | Surcoût heures supplémentaires, embauche évitée, perte productivité experts saturés | `1 FTE évité × 100 000 $ chargé = 100 000 $` (engagement client validé) |
| 17 | Impact $ Dépendances Externes | Coût des blocages externes : SI fournisseur lent, intégration manquante, partenaire en retard | `Pénalités contractuelles 50 000 $/an dues à retards Sabre` (cité par sponsor) |
| 18 | Impact $ Frictions Inter-Départementales | Doublons de travail, allers-retours email, mauvaise communication chiffrés en temps × salaire | `2 FTE × 20 % de leur temps × 75 000 $ = 30 000 $` (chiffre validé en atelier) |

**Coût horaire fully-loaded par défaut** : 80 $/h, profil mixte employé québécois, charges sociales et overhead inclus. Modifiable dans `calculation_rules.md` ou ligne par ligne si le sponsor fournit un taux différent.

### Bloc 5 — Score et confiance (cols 19 à 22)

| # | Colonne | Type | Logique |
|---|---------|------|---------|
| 19 | Impact Total Documenté ($/an) | calculé | SUM(col13 à col18). Reflète uniquement les chiffres durs. |
| 20 | Disponibilité Données | 1-3 + label | voir rubrique |
| 21 | Complétude Chiffrage | 1-3 + label | voir rubrique |
| 22 | Urgence Stratégique | 1-3 + label | voir rubrique |

**Disponibilité Données (col 20)** — basée sur DAMA-DMBOK Data Availability/Accessibility :
- `3 - structurées dans système accessible` : données déjà dans Business Central, CRM, fichier Excel structuré accessible
- `2 - mixtes partiellement accessibles` : une partie dans système, l'autre dispersée (papier, mémoire, multiples fichiers ad hoc)
- `1 - dispersées non structurées` : papier, post-it, mémoire des employés, fichiers personnels non centralisés

**Complétude Chiffrage (col 21)** — reflète la richesse réelle des chiffres durs en cols 13-18, pas la richesse des hypothèses en Notes :
- `3 - toutes sources pertinentes chiffrées` : 4 à 6 cellules sur 6 ont une valeur non-zéro tirée d'un chiffre dur
- `2 - sources principales chiffrées, secondaires manquantes` : 2 à 3 cellules chiffrées, le reste à 0 avec questions structurées en Notes
- `1 - chiffres partiels ou hypothétiques en Notes uniquement` : 0 à 1 cellule chiffrée, l'essentiel de l'impact est en hypothèse en Notes

**Urgence Stratégique (col 22)** — basée sur WSJF Cost of Delay :
- `3 - critique court terme` : OKR explicite du client, fenêtre client ou concurrence, sponsor pousse activement avec une date butoir mentionnée
- `2 - important moyen terme` : sponsor identifié, mention récurrente en atelier, pas de date butoir mais reconnu comme priorité
- `1 - nice-to-have` : mention isolée, pas de sponsor actif, ne fait pas partie des préoccupations principales

### Bloc 6 — Verdict (cols 23 à 25)

| # | Colonne | Type | Logique |
|---|---------|------|---------|
| 23 | Score Priorité Impact | calculé | Voir formule dans calculation_rules.md |
| 24 | Verdict Impact | calculé via grille | Voir grille dans calculation_rules.md |
| 25 | Notes | texte structuré | Sources citées, hypothèses, root cause, section "À chiffrer en atelier", autres notes |

**Notes (col 25)** — structure suggérée :

```
Sources : [citations atelier verbatim, fichier source, date]
Hypothèses : [coût horaire, multiplicateurs, formules utilisées]
Root cause : [si applicable] cette automatisation est la root cause des symptômes
[liste], qui ont été consolidés ici en Step 2bis pour éviter de double-compter.
OU : ce use case est un symptôme dont la root cause est [autre use case].
À chiffrer en atelier :
- [Question précise au sponsor]. À titre indicatif, si [hypothèse], cela
  représenterait environ [valeur] en Impact $ [colonne], soit [+X %] d'Impact
  Total.
- [Autre question].
Autres notes : [conflits entre sources, contexte sponsor, override LLM justifié]
```

### Bloc 7 — Dépendances (col 26)

| # | Colonne | Type | Source |
|---|---------|------|--------|
| 26 | Dépend de | texte libre | Use cases prérequis identifiés en Step 2bis |

**Dépend de (col 26)** :
- Lister les use cases que celui-ci nécessite en amont pour livrer son Impact
- Format : nom du use case prérequis (matchant exactement la col 1 d'une autre ligne pour permettre lookup), virgules pour plusieurs
- Exemple : `Data Lake, Workestimity opérationnel`
- Vide si aucune dépendance externe (cas par défaut pour la majorité des use cases)
- Mentionner les dépendances **non-use-case** (intégration tierce, condition contractuelle, autre projet hors scope IA) en Notes plutôt qu'ici
- Cette colonne **n'entre pas dans le calcul** du Score Priorité Impact ni du Verdict. Elle sert de signal pour la synthèse (section "Dépendances et root causes") et pour aider l'analyste à planifier l'ordre de lancement

---

## Sources méthodologiques détaillées

- **BABOK v3 §10.10 Business Case** : justifie la décomposition de la valeur en sources distinctes (financières directes, opportunités, risques évités, valeur intangible). Ce skill réduit les sources intangibles à des questions en Notes, faute de chiffrage dur disponible en atelier.
- **SAFe WSJF** : Cost of Delay combine User Business Value, Time Criticality et Risk Reduction / Opportunity Enablement. Adapté ici en Urgence Stratégique × Impact Total Documenté × Personnes Affectées. La dimension Job Size (effort) est explicitement retirée et déférée à un futur skill complémentaire. La notion de SAFe Enabler (composant infrastructure qui débloque des features futures) informe la colonne Dépend de.
- **5 Whys de Taiichi Ohno (Toyota Production System) + Ishikawa Diagram de Kaoru Ishikawa** : informent la Step 2bis Root Cause Analysis du workflow. Permettent de distinguer le pain point apparent (symptôme observable en atelier) de la cause profonde actionnable par une automatisation. Sans cette étape, le CSV liste des symptômes en parallèle (ex : "Valérie saturée", "soumissions lentes", "erreurs marge") et triple-compte l'impact d'une seule automatisation possible.
- **DAMA-DMBOK §3 Data Quality Dimensions** : la dimension Availability/Accessibility informe le critère col 20 Disponibilité Données.
- **MIA Innovation pratique terrain** : la règle d'or "chiffres durs uniquement en cellules" vient de l'observation que les LLM gonflent systématiquement les estimations contextuelles. Position de prudence pour préserver la crédibilité de la priorisation auprès des sponsors clients.
