# skills — marketplace agrégée de Samuel Audette

Marketplace Claude Code regroupant les skills publics et stables de Samuel Audette. Une seule commande pour les avoir tous, tu actives ceux que tu veux.

Pour les versions instables, expérimentales, archives ou forks adaptés, voir la marketplace beta : [`RunLittleTurtle/skills-beta`](https://github.com/RunLittleTurtle/skills-beta).

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

### Flow d'usage — Product Management → Directeur de compte

```mermaid
flowchart LR
  C[scenario-uc<br/><i>Product Management</i>] --> D[product-brief<br/><i>Product Management</i>]
  C --> E[mermaid-flow<br/><i>Directeur de compte</i>]
```

- **scenario-uc** : formalise un use case en scénario PRD avec diagramme de séquence Mermaid.
- **product-brief** : transforme des inputs hétérogènes (BA, transcripts, OKRs) en product brief one-pager au format PRD Authentik.
- **mermaid-flow** : vulgarise un scénario pour un Directeur de compte ou stakeholder non technique.

Les autres skills (`coordination`, `skill-creator-turtle`, `bug-us-mapping`) sont indépendants de ce flow.

### Tableau récapitulatif

| Nom | Description |
|---|---|
| `bug-us-mapping` | Croise un export CSV de User Stories Jira avec un export CSV de bugs pour identifier les US Done qui ne sont pas réellement complétées à 100%. v1.1.0 : auto-détection avec validation interactive, mapping sémantique strict, table unique triée par % complétion croissant (30/50/70/85/100), section bugs orphelins avec validation PO. |
| `coordination` | Coordonne plusieurs instances Claude qui travaillent en parallèle sur le même repo via locks markdown. v1.1.0 : auto-détection git/folder, setup conditionnel (`.gitignore` + note `CLAUDE.md` seulement si pertinent). |
| `mermaid-flow` | Transforme un flow (texte, fichier markdown, mermaid existant ou image) en flowchart Mermaid simplifié pour personnes peu techniques (max 10 étapes, palette pastel light-mode, emojis acteurs 👤🤖⚙️🖥️⚖️). |
| `product-brief` | Transforme inputs hétérogènes (notes BA, data points, transcripts, insights discovery, OKRs) en product brief one-pager au format PRD Authentik. v2 : posture Product Manager Senior, scan préliminaire de la doc, validation par section, citations verbatim complètes, AARRR conditionnel. |
| `scenario-uc` | Transforme tout input (md, PDF, image, URL Drive, idée verbale) en scénario use-case au format PRD Authentik avec diagramme de séquence Mermaid. Sortie en français. |
| `skill-creator-turtle` | Meta-skill générique pour créer un nouveau skill dans ta propre marketplace personnelle (3 cibles : marketplace, standalone, autre outil). Renommé de `skill-creator` pour ne pas se confondre avec le skill-creator officiel d'Anthropic. |

---

## Marketplace beta

Pour les versions en cours de développement, parallèles, archives ou forks adaptés, voir [`RunLittleTurtle/skills-beta`](https://github.com/RunLittleTurtle/skills-beta) :

```
/plugin marketplace add RunLittleTurtle/skills-beta
```

Skills actuellement disponibles en beta : `agent-talk-beta`, `product-brief-v1-beta`, `product-management`, `scenario-uc-v2-beta`, `skill-creator-turtle-beta`, `use-case-prioritization-beta`, `use-case-value-beta`.

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

Installe le skill `skill-creator-turtle` (`/plugin install skill-creator-turtle@skills`) et invoque-le. Il te guide interactivement et met à jour ce repo automatiquement.

Pour expérimenter avec une version refondue qui ajoute la modification de skills existants, utilise [`skill-creator-turtle-beta`](https://github.com/RunLittleTurtle/skills-beta/tree/main/plugins/skill-creator-turtle-beta) dans la marketplace beta.

---

## Licence

MIT — voir [LICENSE](./LICENSE).
