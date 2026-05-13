# use-case-prioritization (v2.1)

> Scorer, prioriser et chiffrer des use cases d'AI/automatisation à partir de matériel de découverte de processus (photos de post-its, transcripts d'ateliers, inventaires CSV de 500+ lignes, notes de réunion). Combine BABOK Business Case, UiPath Suitability sur **échelle 1-3** avec **labels qualitatifs intégrés dans les cellules**, coûts fully-loaded, **Run cost benchmarké par recherche web** selon le type d'agent, et **score de confiance hybride** (complétude × validation LLM) qui évite que toutes les lignes affichent faussement 100%. Sortie : CSV de travail (38 colonnes v5) + synthèse exécutive en français Québec.

## Use case

Tu as fini un atelier de découverte (post-its, transcript, inventaire de processus) et tu dois décider **quel use case scoper en premier**, construire une **enveloppe budgétaire** crédible, ou faire le **triage d'une pipeline d'automatisation**. Le skill transforme le matériel brut en deux livrables prêts à partager : un CSV 37 colonnes pour l'analyste fonctionnel (avec lignes FORMULES et SOURCE pour traçabilité) et une synthèse markdown pour les directeurs de compte (Top 5 + sections « À éliminer » / « À retravailler »).

## Nouveautés v2 (vs v1) et v2.1 (vs v2)

**v2.0** :
- **Échelles 1-5 → 1-3** sur tous les critères qualitatifs (Suitability, Risque). Granularité plus humaine en atelier.
- **Format cellule `<chiffre> - <label>`** : la cellule contient à la fois le chiffre (pour la math) ET le label qualitatif (pour la lisibilité humaine). Exemple : `3 - récurrent standardisé`.
- **Run cost benchmarké LLM** : remplace l'heuristique `Build × 15%` par `Volume Qté × Coût Unitaire Agent`, où le $/run est récupéré par recherche web ciblée (a16z, Menlo, pricing officiels OpenAI/Anthropic) en avant-dernière étape. Source et date citées dans Notes.
- **8 colonnes supprimées** (Nb Intégrations, Volume Annuel unité, Personnes Impactées, Notes Architecte fusionnée dans Notes, Nb Colonnes Critiques, Qualité Source, Cohérence Check, Confiance Globale composite) ; **2 nouvelles** (Type Agent, Coût Unitaire Agent).

**v2.1** :
- **Nouvelle col 35 `Validation LLM`** (`1 - estimation contextuelle` / `2 - mixte` / `3 - source primaire validée`) : le LLM s'auto-évalue ligne par ligne sur la fraction de valeurs issues de la source vs estimées depuis le contexte. Justification obligatoire dans Notes.
- **Verdict Confiance hybride** : `Confiance Globale = Confiance Données × Validation_Multiplier` (1.0 / 0.75 / 0.5). Évite que toutes les lignes saturent à 100% quand le LLM remplit tous les slots avec des estimations. Sur le CSV Faspac réel, ramène les 14 lignes à 100% à des Confiance Globale différenciées entre 50% et 100%.
- Total : **38 colonnes** au lieu de 37.

## Installation

### Pour Claude Code

```
/plugin marketplace add RunLittleTurtle/skills
/plugin install use-case-prioritization@skills
```

Mise à jour : `/plugin marketplace update skills`. Désinstallation : `/plugin uninstall use-case-prioritization@skills`.

### Pour les autres outils (Claude Desktop, OpenCode, Cursor, GitHub Copilot, Gemini CLI, Goose, etc.)

Standard ouvert [agentskills.io](https://agentskills.io). Clone le repo et copie le skill dans le dossier skills de ton outil :

```bash
git clone https://github.com/RunLittleTurtle/skills.git
cp -R skills/plugins/use-case-prioritization/skills/use-case-prioritization <DOSSIER_SKILLS_DE_TON_OUTIL>/
```

| Outil | Dossier cible |
|---|---|
| Claude Code | `~/.claude/skills/` |
| Claude Desktop (macOS) | `~/Library/Application Support/Claude/skills/` |
| OpenCode | `~/.opencode/skills/` |
| Cursor | voir [docs Cursor skills](https://cursor.com/docs/context/skills) |
| GitHub Copilot | voir [docs Copilot agent skills](https://docs.github.com/en/copilot/concepts/agents/about-agent-skills) |

## Invocation

Slash command Claude Code : `/use-case-prioritization:use-case-prioritization` (ou `/use-case-prioritization` selon ta config).

Le skill détecte les mots-clés courants : `prioriser`, `prioritization`, `use case`, `ROI`, `payback`, `business case`, `suitability`, `scoping`, `enveloppe budgétaire`, `quel use case en premier`, `automation pipeline`, `process triage`, `atelier découverte`, `post-its`, `cartographie processus`, `chiffrer use case`.

## Contenu

- `skills/use-case-prioritization/SKILL.md` — workflow complet (frontmatter `name` + `description` + étapes).
- `skills/use-case-prioritization/references/framework_v5.md` — explication des 38 colonnes v5, sources méthodologiques (BABOK, UiPath, SAFe, DAMA-DMBOK).
- `skills/use-case-prioritization/references/calculation_rules.md` — formules v5, valeurs par défaut, seuils de verdict, table de fallback benchmark.
- `skills/use-case-prioritization/references/csv_template.csv` — gabarit CSV 37 colonnes avec lignes FORMULES, SOURCE et exemple use case fictif.
- `skills/use-case-prioritization/references/exec_summary_template.md` — gabarit de synthèse exécutive en français Québec.

## Licence

MIT — voir [LICENSE](../../LICENSE) à la racine du repo.
