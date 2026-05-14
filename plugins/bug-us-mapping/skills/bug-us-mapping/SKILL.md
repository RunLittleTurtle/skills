---
name: bug-us-mapping
description: Analyser l'écart entre User Stories en statut "Done" et bugs ouverts dans un export CSV Jira, pour identifier les US qui ne sont pas réellement complétées à 100%. Auto-détection des fichiers CSV avec validation interactive obligatoire, mapping sémantique strict bugs ↔ US (pas de liens spéculatifs), production d'une seule table markdown triée du moins complet au plus complet (% croissant). Échelle de complétion 30/50/70/85/100. Sortie en français Québec sans em-dash. À invoquer quand l'utilisateur dispose d'exports Jira (US + bugs au format CSV) et veut savoir quelles US Done ont des bugs ouverts qui révèlent une lacune fonctionnelle, ou pour construire une roadmap de finition par % complétion.
---

# bug-us-mapping - Analyse de liaison Bugs ↔ User Stories Jira

Skill qui croise un export CSV de User Stories Jira avec un export CSV de bugs Jira pour identifier les US qui ne sont pas réellement complétées à 100% (même si en statut "Done"), via un mapping sémantique strict. Produit une table markdown unique triée par % complétion croissant.

**Réponds en français Québec. Sois concis. Pas d'em-dash (utilise tiret court -). Pas de symbole §. Valide TOUJOURS avec l'utilisateur via AskUserQuestion avant de produire le livrable final.**

## Étape 1 - Détection auto + validation des CSV

Cherche dans le dossier courant et sous-dossiers (profondeur 2 max) les fichiers CSV qui ressemblent à des exports Jira :

```bash
find . -maxdepth 2 -type f -iname "*.csv" \( -iname "*bug*" -o -iname "*us*" -o -iname "*story*" -o -iname "*jira*" -o -iname "*backlog*" \) 2>/dev/null
```

Pour chaque fichier candidat, affiche :
- Nom + chemin relatif
- Nombre de lignes (`wc -l`)
- Première ligne d'en-tête (truncated)

**OBLIGATOIRE** : valide via `AskUserQuestion` les 2 fichiers à utiliser, MÊME SI la détection semble évidente :

> "Voici les CSV détectés. Confirme lequel est le fichier des US (Stories) et lequel est celui des Bugs."

Si rien trouvé, demande directement les 2 chemins. Si plusieurs candidats par type, présente-les en options et laisse l'utilisateur choisir (avec option "Other" pour chemin custom).

## Étape 2 - Validation des colonnes Jira

Les exports Jira varient selon la configuration du projet. Lis l'en-tête de chaque CSV et identifie les colonnes essentielles :

- **US CSV** : `Summary`, `Issue key`, `Issue Type`, `Status`, `Parent key`, `Parent summary`, `Description`, éventuellement `Inward issue link (...)`, `Outward issue link (...)`
- **Bugs CSV** : mêmes colonnes essentielles

Si le naming est ambigu (langue différente, colonnes custom), confirme via `AskUserQuestion` les colonnes utilisées. Détecte la langue du projet à partir des valeurs de `Status` et `Issue Type` (FR/EN/autre) pour adapter le filtrage.

## Étape 3 - Extraction et filtrage

Filtre :
- **US** : lignes où `Issue Type` est dans {`Story`, `User Story`, équivalent localisé}
- **Bugs** : lignes où `Issue Type == Bug` (cela inclut TOUS les tickets typés Bug dans Jira, peu importe leur préfixe titre `[BUG]`, `[AMÉLIORATION]`, `[À investiguer]`, `[ON HOLD]`, etc.)

Demande via `AskUserQuestion` si l'utilisateur veut filtrer les US par statut (par défaut, garder TOUTES les US ; option recommandée : ne garder que `Done` car ce sont elles dont on remet la complétion en question).

Affiche un récap : `N US extraites, M bugs extraits` avant de continuer.

## Étape 4 - Analyse sémantique STRICTE (coeur du skill)

Pour chaque bug, identifie les US dont le périmètre couvre **DIRECTEMENT** le bug. Croise :

- **Source de donnée touchée** (déduite du titre + description du bug) ↔ source couverte par la US
- **Capacité fonctionnelle** mise en défaut ↔ capacité prévue par la US
- **Acteur** concerné (rôle interne/externe, conseillère/client, etc.)

**RÈGLE D'OR** : un bug n'est rattaché à une US que si le lien est de confiance **Haute** - la US couvre directement le périmètre du bug. PAS de liens "Moyenne" ou "Faible" ou par association lointaine. Mieux vaut sous-rattacher que sur-rattacher.

Exemples de bons liens (Haute confiance) :
- Bug "Bot retourne 25 pieds au lieu de 25,8 pieds pour véhicule X" ↔ US "Accès aux informations sur les véhicules" (Haute)
- Bug "Numéro réservation fournisseur absent du RAG" ↔ US "Accès aux bons d'échange (Vouchers)" (Haute)

Exemples de liens à REJETER (faibles, à ne pas inclure) :
- Bug "Mauvaise priorité de source" ↔ US "Traçabilité des sources" (la traçabilité elle-même fonctionne, c'est la priorisation qui est mauvaise - lien indirect, à rejeter)
- Bug "Info périmée 2022" ↔ US "Metadata de mise à jour" (le bug est sur le contenu, pas sur la date - lien indirect)

Lis attentivement les `Description` et `Critères d'acceptation` des US (souvent dans des panels `{panel:bgColor=...}`) pour ne pas se tromper sur le périmètre réel.

Un bug peut être rattaché à plusieurs US si chacune couvre directement un aspect distinct du bug (ex: bug avec 2 problèmes distincts, chacun couvert par une US différente).

## Étape 5 - Calcul du % complétion

| % | Statut qualitatif | Critère |
|---|------------------|---------|
| 100% | OK | Aucun bug lié |
| 85% | Bugs mineurs | 1 bug d'UX/format/cosmétique |
| 70% | Bugs mineurs | 2 bugs mineurs OU 1 bug de précision de donnée OU 1 bug ON HOLD |
| 50% | Bugs majeurs | 1 bug de priorisation source / réponse fausse partielle |
| 30% | Bugs majeurs | Plusieurs bugs majeurs OU fonctionnalité retourne info fausse |
| <20% | Non fonctionnel | Fonctionnalité cassée / non implémentée |

Pour estimer la sévérité d'un bug :
- **Mineur** : UX, format, cosmétique, amélioration de présentation
- **Précision donnée** : valeur incorrecte mais proche, métadonnée manquante
- **Majeur** : réponse fausse fonctionnelle, priorisation incorrecte de source, fonctionnalité contournée

## Étape 6 - Tri + génération de la table markdown

Colonnes obligatoires :

| US | Titre | Parent | Bugs liés | % Complétion | Commentaire |
|----|-------|--------|-----------|--------------|-------------|

Tri **croissant par % Complétion** : 30% en haut, puis 50%, 70%, 85%, 100% en bas. Les éventuels "hors scope" tout en bas avec `n/a` dans la colonne %.

Format du `Commentaire` : 1 phrase concise qui résume QUOI est cassé (pas le pourquoi du mapping, pas la justification du lien).

Format des `Bugs liés` : liste séparée par virgules des `Issue key` des bugs (ex: `AC-411, AC-413`).

## Étape 7 - Validation du chemin de sortie + écriture

Propose un chemin par défaut :
1. Même dossier que les CSV détectés (priorité 1)
2. Sinon dossier courant
3. Nom par défaut : `bug_us_mapping.md`

Valide via `AskUserQuestion` :

> "Où veux-tu sauvegarder le fichier de mapping ? Défaut : `<chemin>`"

Écris le fichier avec :
- Titre `# Mapping Bugs ↔ User Stories - <nom du projet détecté>`
- Section `## Contexte` : nombre d'US, nombre de bugs, dates des bugs (min/max), source des CSV
- Section `## Tableau` : la table unique triée

Pas de section "synthèse" ou "recommandations" séparées dans le fichier (ces éléments vont dans le récap conversation).

## Étape 8 - Récap dans la conversation

Affiche dans le chat (PAS dans le fichier) :
- Nombre d'US avec bugs / sans bugs / hors scope
- **Top 5 US à retravailler** (les premières par % croissant) sous forme de mini-table
- **Insight transversal** : si plusieurs bugs partagent une cause racine commune (ex: priorisation de source, ton, format), mentionne-le en 1-2 phrases. Cette analyse aide l'utilisateur à prioriser un sous-epic correctif plutôt que des fixes ponctuels.

## Notes importantes pour Claude pendant l'exécution

- **Ne hardcode JAMAIS** de chemins absolus, de noms de projet (ex: "Authentik"), de préfixes Jira (ex: "AC-"), de terminologie métier spécifique. Le skill doit fonctionner pour tout projet Jira.
- **Toujours valider** via AskUserQuestion les étapes critiques (fichiers détectés, colonnes ambiguës, chemin de sortie), même si la détection semble évidente.
- **Strict mode pour le mapping** : si un lien te semble "moyen" ou "faible", ne l'inclus PAS. Mieux vaut une table propre avec quelques US à 100% en trop qu'une table polluée par des liens spéculatifs.
- **Format strict** : une seule table, triée par % croissant, colonnes exactes.
- **Pas d'em-dash** dans le SKILL.md ni dans les outputs (utilise `-` court). Pas de `§` (écris "section X").
- **Si les exports CSV ont des descriptions très longues** (panels Jira, contenu RAG inclus), tronque mentalement à 800 caractères pour l'analyse sémantique (le périmètre/intent suffit, pas besoin du contenu intégral).
- **Couverture finale** : vérifie que chaque bug du CSV est référencé au moins une fois dans le mapping. Si un bug n'a aucune US directe, mentionne-le explicitement dans le récap conversation comme "bug sans US correspondante - candidat à création de nouvelle US".
