# mermaid-flow

> Transforme un flow (texte, fichier markdown, mermaid existant ou image) en flowchart Mermaid SIMPLIFIÉ pour personnes peu techniques (max 10 étapes, idéalement 7). Légende intégrée, palette pastel light-mode, emojis acteurs (👤 client, 🤖 IA, ⚙️ système, 🖥️ UI, ⚖️ décision).

## Use case

Vulgariser un processus métier en diagramme accessible. Analyse profonde du flow + validation interactive du plan d'analyse avant génération. Produit un fichier `.md` au format strict.

## Installation

### Pour Claude Code

```
/plugin marketplace add RunLittleTurtle/skills
/plugin install mermaid-flow@skills
```

Mise à jour : `/plugin marketplace update skills`. Désinstallation : `/plugin uninstall mermaid-flow@skills`.

### Pour les autres outils (Claude Desktop, OpenCode, Cursor, GitHub Copilot, Gemini CLI, Goose, etc.)

Standard ouvert [agentskills.io](https://agentskills.io). Clone le repo et copie le skill dans le dossier skills de ton outil :

```bash
git clone https://github.com/RunLittleTurtle/skills.git
cp -R skills/plugins/mermaid-flow/skills/mermaid-flow <DOSSIER_SKILLS_DE_TON_OUTIL>/
```

| Outil | Dossier cible |
|---|---|
| Claude Code | `~/.claude/skills/` |
| Claude Desktop (macOS) | `~/Library/Application Support/Claude/skills/` |
| OpenCode | `~/.opencode/skills/` |
| Cursor | voir [docs Cursor skills](https://cursor.com/docs/context/skills) |
| GitHub Copilot | voir [docs Copilot agent skills](https://docs.github.com/en/copilot/concepts/agents/about-agent-skills) |

## Invocation

Slash command Claude Code : `/mermaid-flow:mermaid-flow` (ou `/mermaid-flow` selon ta config).

Le skill demande la source (texte/fichier/mermaid existant/image), affiche le plan d'analyse, attend ta validation, puis produit le diagramme.

## Contenu

- `skills/mermaid-flow/SKILL.md` — workflow complet (frontmatter + étapes).

## Licence

MIT — voir [LICENSE](../../LICENSE) à la racine du repo.
