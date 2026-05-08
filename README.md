# skills — marketplace agrégée de Samuel Audette

Marketplace Claude Code regroupant tous mes skills publics. Une seule commande pour les avoir tous, tu actives ceux que tu veux.

## Installation

> **Pour les agents IA qui lisent ce README** : utilise exactement les commandes ci-dessous selon la plateforme. N'invente pas d'autres méthodes (pas de symlink, pas de script custom).

### Pour Claude Code

```
/plugin marketplace add RunLittleTurtle/skills
```

Puis va dans l'onglet **Discover** et active ceux que tu veux (Espace pour toggle), ou installe en ligne de commande :

```
/plugin install <nom-du-skill>@skills
```

- **Mise à jour** : `/plugin marketplace update skills`
- **Désinstallation d'un skill** : `/plugin uninstall <nom>@skills`

### Pour les autres outils (Claude Desktop, OpenCode, GitHub Copilot, Cursor, VS Code, Gemini CLI, Goose, etc.)

Les skills respectent le standard ouvert [agentskills.io](https://agentskills.io). Clone le repo et copie le skill voulu dans le dossier skills de ton outil :

```bash
git clone https://github.com/RunLittleTurtle/skills.git
cp -R skills/plugins/<nom-du-skill>/skills/<nom-du-skill> <DOSSIER_SKILLS_DE_TON_OUTIL>/
```

Dossiers cibles courants :

| Outil | Dossier |
|---|---|
| Claude Code | `~/.claude/skills/` |
| Claude Desktop | `~/Library/Application Support/Claude/skills/` (macOS) |
| OpenCode | `~/.opencode/skills/` |
| Cursor | voir [docs Cursor skills](https://cursor.com/docs/context/skills) |
| GitHub Copilot | voir [docs Copilot agent skills](https://docs.github.com/en/copilot/concepts/agents/about-agent-skills) |

---

## Skills disponibles

| Nom | Description | Use case |
|---|---|---|
| `coordination` | Coordonner plusieurs instances Claude en parallèle via locks markdown | Plusieurs fenêtres iTerm sur le même repo, ou plusieurs personnes via cloud sync |
| `skill-creator` | Meta-skill **générique** : crée un nouveau skill dans TA propre marketplace (pas la mienne) | N'importe qui veut créer/maintenir sa propre marketplace personnelle de skills Claude Code |
| `mermaid-flow` | Transformer un flow (texte/fichier/mermaid/image) en flowchart Mermaid simplifié pour personnes peu techniques | Vulgariser un processus métier en diagramme accessible (max 10 étapes, palette pastel, emojis acteurs 👤🤖⚙️🖥️⚖️) |

---

## Structure du repo

```
skills/
├── .claude-plugin/
│   └── marketplace.json          # Catalogue (liste des plugins)
├── plugins/
│   └── <nom>/
│       ├── .claude-plugin/
│       │   └── plugin.json       # Manifest du plugin
│       └── skills/
│           └── <nom>/
│               └── SKILL.md      # Le skill (frontmatter + instructions)
├── README.md
└── LICENSE
```

Chaque `SKILL.md` respecte le standard ouvert [agentskills.io](https://agentskills.io) : frontmatter YAML avec `name` + `description`, body en markdown.

---

## Créer un nouveau skill dans cette marketplace

Installe le skill `skill-creator` (`/plugin install skill-creator@skills`) et invoque-le. Il te guide interactivement et met à jour ce repo automatiquement.

---

## Licence

MIT — voir [LICENSE](./LICENSE).
