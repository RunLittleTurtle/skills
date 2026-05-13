# Template de synthèse exécutive

Ce template définit la structure du fichier `synthese_priorisation.md`. Suivre exactement cette structure, en remplaçant les placeholders entre crochets.

## Règles de production

- **Langue**: Français Québec
- **Aucun em dash** (utiliser virgules, parenthèses, deux-points)
- **2-3 lignes maximum** par use case dans le top priorités
- **Maximum 5 use cases** dans le top priorités
- **Tri**: par Score Priorité décroissant, mais en filtrant d'abord par Verdict ROI = "Quick win" ou "Go" ou "A challenger"
- **Section "À éliminer"**: tous les use cases avec Verdict ROI = "Pass"
- **Section "À retravailler"**: tous les use cases avec Verdict Confiance < 60% qui ne sont ni dans top ni dans à éliminer

## Structure à reproduire

```markdown
# Synthèse de priorisation des use cases AI

**Date d'analyse**: [date du jour AAAA-MM-JJ]
**Source des inputs**: [type de source: post-its + transcript / table de N processus / etc.]
**Use cases évalués**: [nombre total]
**Recommandés pour scoping**: [nombre avec Verdict ROI Go ou Quick win, Confiance ≥ 60%]

## Top priorités pour scoping

[Pour chaque use case retenu, dans cet ordre, max 5]

### 1. [Nom du Use Case]

**Sponsor**: [nom ou "À identifier"] | **Département**: [département]

**Snapshot**: Suitability [XX/100], Confiance [XX%], Payback [X mois], ROI An 1 [XX%], Bénéfice net [XX k$/an], Run [XX $/an basé sur $X.XX/run]

**Pourquoi prioritaire** (2-3 lignes): [Synthétiser pourquoi ce use case mérite d'être scopé en premier. Mentionner les signaux forts: volume confirmé, sponsor engagé, données disponibles, alignement stratégique, ou pertes évitées tangibles.]

**À valider avant scoping** (si applicable, 1 ligne): [Mentionner 1-2 hypothèses fortes ou inputs manquants si Confiance < 80%. Sinon omettre cette ligne.]

---

### 2. [Nom du Use Case]
[Même structure]

[etc. jusqu'à 5 maximum]

## À éliminer (Verdict ROI = Pass)

[Liste à puces, 1 ligne par use case]

- **[Nom]**: [raison brève: payback trop long, ROI négatif structurel, volume trop bas, etc.]
- **[Nom]**: [raison]

## À retravailler avant décision (Confiance < 60%)

[Liste à puces, 1-2 lignes par use case]

- **[Nom]** (Confiance [XX%]): [ce qui manque pour décider. Ex: volumes à confirmer, sponsor à identifier, complexité technique à évaluer avec architecte.]

## Méthodologie

Cette priorisation combine:
- **Suitability Score** (UiPath / Deloitte): filtre qualitatif sur 8 critères notés 1-3 avec labels qualitatifs intégrés (volume, règles, données, stabilité, exceptions, impact, alignement, sponsorship)
- **Business case quantitatif** (BABOK 10.10): bénéfice net annuel basé sur volume × temps × coût horaire, avec haircut de réalisation (Prosci 50-70%)
- **Effort architecte** (Agile Estimation Cohn): T-shirt sizing converti via barème agence
- **Run cost benchmarké** (v5): Volume Qté × Coût Unitaire Agent, où le $/run est récupéré par recherche web sur sources fiables (a16z, Menlo, pricing officiels) avec source et date citées
- **Score de confiance par complétude** (DAMA-DMBOK): pourcentage des 20 colonnes critiques remplies

**Seuils de décision**:
- Payback < 6 mois = Quick win, 6-12 mois = Go, 12-18 = A challenger, 18-24 = Stratégique seulement, > 24 = Pass
- Confiance ≥ 80% = Fiable, 60-79% = A valider 1-2 points, 40-59% = Hypothèses fortes, < 40% = Atelier requis

**Sources benchmark Run** utilisées dans cette priorisation:
[Lister les sources citées par use case, avec date de consultation. Ex: "a16z State of AI 2026 pour email auto $0.04/run, consulté 2026-05-13"]

Pour le détail complet des 37 colonnes et formules, voir le CSV de priorisation et le framework de référence v5.
```

## Notes sur les cas particuliers

**Si aucun use case ne passe le filtre (tout est en Pass ou Confiance trop basse)**:
Remplacer la section "Top priorités" par un message:
```
## Aucun use case ne passe les filtres actuels

Avec les inputs fournis, aucun use case n'atteint à la fois un Payback < 24 mois et une Confiance ≥ 60%. Recommandations:

1. [Use case avec le meilleur score parmi les "À retravailler"]: ce serait le candidat principal si [élément manquant] était validé.
2. Organiser un atelier de chiffrage avec [stakeholders pertinents] pour collecter [données manquantes].
```

**Si un seul use case est fourni**:
Pas de section "Top priorités" en numéroté. Faire une analyse approfondie en une seule fiche détaillée.

**Si l'input ne permet pas de calculer le ROI (volumes manquants partout)**:
Produire la synthèse en se basant sur Suitability + Confiance seulement. Documenter clairement que le ROI ne peut être calculé sans les volumes, et lister les données à collecter en priorité.
