# scenario-uc

> Transforme n'importe quel input (texte, brouillon md, PDF, image, URL Drive, idée verbale) en un fichier markdown de scénario use-case au format Authentik PRD : description, intent, séquence numérotée, diagramme de séquence Mermaid. Analyse profonde + validation interactive avant génération. Sortie en français.

## Use case

Formaliser un cas d'utilisation produit (description avec acteurs typés + intent + séquence numérotée + `sequenceDiagram`) à partir de n'importe quelle source — texte brut, brouillon, PDF, screenshot de whiteboard, URL Google Drive, conversation. Pensé pour le Projet 4 Authentik et tout PRD du même style.

## Installation

### Pour Claude Code

```
/plugin marketplace add RunLittleTurtle/skills
/plugin install scenario-uc@skills
```

Mise à jour : `/plugin marketplace update skills`. Désinstallation : `/plugin uninstall scenario-uc@skills`.

### Pour les autres outils (Claude Desktop, OpenCode, Cursor, GitHub Copilot, Gemini CLI, Goose, etc.)

Standard ouvert [agentskills.io](https://agentskills.io). Clone le repo et copie le skill dans le dossier skills de ton outil :

```bash
git clone https://github.com/RunLittleTurtle/skills.git
cp -R skills/plugins/scenario-uc/skills/scenario-uc <DOSSIER_SKILLS_DE_TON_OUTIL>/
```

| Outil | Dossier cible |
|---|---|
| Claude Code | `~/.claude/skills/` |
| Claude Desktop (macOS) | `~/Library/Application Support/Claude/skills/` |
| OpenCode | `~/.opencode/skills/` |
| Cursor | voir [docs Cursor skills](https://cursor.com/docs/context/skills) |
| GitHub Copilot | voir [docs Copilot agent skills](https://docs.github.com/en/copilot/concepts/agents/about-agent-skills) |

## Invocation

Slash command Claude Code : `/scenario-uc:scenario-uc` (ou `/scenario-uc` selon ta config).

Le skill demande l'input puis valide interactivement le plan d'analyse avant de générer le fichier.

## Contenu

- `skills/scenario-uc/SKILL.md` — workflow complet (frontmatter `name` + `description` + étapes).

## Licence

MIT — voir [LICENSE](../../LICENSE) à la racine du repo.
