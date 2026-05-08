# coordination

> Coordonne plusieurs instances Claude qui travaillent en parallèle sur le même repo (typiquement plusieurs fenêtres iTerm). Crée ou rejoint un fichier de log par focus (sprint, task, prd, etc.) avec un système de locks logiques en markdown — pour éviter que deux instances écrasent leurs édits sur les mêmes fichiers de doc.

## Use case

Tu lances Claude dans deux ou trois fenêtres iTerm sur le même repo (sprints, PRD, user stories). Sans coordination, deux instances peuvent toucher au même fichier en parallèle et perdre du travail. Ce skill installe un protocole d'**advisory locks** simple (fichiers `COORDINATION.<FOCUS>.md`) que chaque instance lit avant d'éditer un fichier partagé.

Lecture/recherche/planification : libre. Edit/Write sur fichier partagé : ligne `[LOCKED]` obligatoire.

## Installation

### Pour Claude Code

```
/plugin marketplace add RunLittleTurtle/skills
/plugin install coordination@skills
```

Mise à jour : `/plugin marketplace update skills`. Désinstallation : `/plugin uninstall coordination@skills`.

### Pour les autres outils (Claude Desktop, OpenCode, Cursor, GitHub Copilot, Gemini CLI, Goose, etc.)

Standard ouvert [agentskills.io](https://agentskills.io). Clone le repo et copie le skill dans le dossier skills de ton outil :

```bash
git clone https://github.com/RunLittleTurtle/skills.git
cp -R skills/plugins/coordination/skills/coordination <DOSSIER_SKILLS_DE_TON_OUTIL>/
```

| Outil | Dossier cible |
|---|---|
| Claude Code | `~/.claude/skills/` |
| Claude Desktop (macOS) | `~/Library/Application Support/Claude/skills/` |
| OpenCode | `~/.opencode/skills/` |
| Cursor | voir [docs Cursor skills](https://cursor.com/docs/context/skills) |
| GitHub Copilot | voir [docs Copilot agent skills](https://docs.github.com/en/copilot/concepts/agents/about-agent-skills) |

## Invocation

Au début de chaque session, dans chaque fenêtre Claude qui doit participer :

```
/coordination:coordination
```

Le skill détecte si le repo est déjà setup (sinon il propose la phase setup : `COORDINATION.README.md` + `.gitignore` + note dans `CLAUDE.md`), puis te propose de **créer un nouveau focus** (ex: `SPRINT`, `TASK`, `PRD`) ou de **rejoindre** un focus existant. Une fois connecté, l'instance applique le protocole de locks pour le reste de la session.

## Statuts d'une entrée de log

| Statut | Signification |
|---|---|
| `[INTENT]` | Planifié, pas commencé |
| `[LOCKED]` | En train d'écrire MAINTENANT (TTL 30min par défaut) |
| `[WAITING]` | Veut écrire mais bloqué par un `[LOCKED]` actif |
| `[DONE]` | Terminé |
| `[CANCELLED]` | Abandonné |

Détails complets du protocole dans `skills/coordination/SKILL.md`.

## Contenu

- `skills/coordination/SKILL.md` — workflow complet (setup, création/rejoint focus, application du protocole).

## Licence

MIT — voir [LICENSE](../../LICENSE) à la racine du repo.
