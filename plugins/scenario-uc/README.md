# scenario-uc

> Transforme n'importe quel input (texte, brouillon md, PDF, image, URL Drive, idée verbale) en un fichier markdown de scénario use-case au format Authentik PRD complet : description, intent, séquence textuelle numérotée avec séquences alternatives HEC (suffixes a/b/c) et boucles LOOP/FIN LOOP, diagramme de séquence Mermaid avec titre frontmatter et bloc `loop ... end` natif, préfixe AS-IS / TO-BE dans le H1. Analyse profonde + validation interactive renforcée avant génération. Sortie en français.

## Use case

Formaliser un cas d'utilisation produit (description avec acteurs typés + intent + séquence numérotée avec alternatives HEC et boucles + `sequenceDiagram` Mermaid avec `loop ... end`) à partir de n'importe quelle source — texte brut, brouillon, PDF, screenshot de whiteboard, URL Google Drive, conversation. Pensé pour le Projet 4 Authentik et tout PRD du même style. Préfixe AS-IS / TO-BE dans le H1 pour différencier l'analyse du processus existant de la solution future.

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

Le skill demande l'input puis valide interactivement le plan d'analyse (avec questions bloquantes sur AS-IS/TO-BE, alternatives et boucles si applicable) avant de générer le fichier.

## Version

v2.0.0 — ajoute mode AS-IS / TO-BE obligatoire, séquences alternatives HEC (suffixes lettrés a/b/c avec convention de retour), boucles `LOOP : <condition> / FIN LOOP` alignées avec un bloc `loop ... end` Mermaid de même condition, titre du diagramme via frontmatter Mermaid, validation interactive renforcée. La version 1 originale est archivée comme [`scenario-uc-v1-beta`](https://github.com/RunLittleTurtle/skills-beta/tree/main/plugins/scenario-uc-v1-beta) dans la marketplace beta.

## Contenu

- `skills/scenario-uc/SKILL.md` — workflow complet (frontmatter `name` + `description` + étapes).

## Licence

MIT — voir [LICENSE](../../LICENSE) à la racine du repo.
