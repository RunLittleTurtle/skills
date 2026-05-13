# use-case-prioritization

> Scorer, prioriser et chiffrer des use cases d'AI/automatisation à partir de matériel de découverte de processus (photos de post-its, transcripts d'ateliers, inventaires CSV de 500+ lignes, notes de réunion). Combine BABOK Business Case, UiPath Suitability, coûts fully-loaded et ROI pondéré par la confiance. Sortie : CSV de travail (43 colonnes v4) + synthèse exécutive en français Québec.

## Use case

Tu as fini un atelier de découverte (post-its, transcript, inventaire de processus) et tu dois décider **quel use case scoper en premier**, construire une **enveloppe budgétaire** crédible, ou faire le **triage d'une pipeline d'automatisation**. Le skill transforme le matériel brut en deux livrables prêts à partager : un CSV 43 colonnes pour l'analyste fonctionnel (avec lignes FORMULES et SOURCE pour traçabilité) et une synthèse markdown pour les directeurs de compte (Top 5 + sections « À éliminer » / « À retravailler »).

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
- `skills/use-case-prioritization/references/framework_v4.md` — explication des 43 colonnes, sources méthodologiques (BABOK, UiPath, SAFe, DAMA-DMBOK).
- `skills/use-case-prioritization/references/calculation_rules.md` — formules exactes, valeurs par défaut, seuils de verdict.
- `skills/use-case-prioritization/references/csv_template.csv` — gabarit CSV 43 colonnes avec lignes FORMULES et SOURCE.
- `skills/use-case-prioritization/references/exec_summary_template.md` — gabarit de synthèse exécutive en français Québec.

## Licence

MIT — voir [LICENSE](../../LICENSE) à la racine du repo.
