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
| `scenario-uc` | Transformer tout input (md/PDF/image/URL Drive/idée) en scénario use-case au format Authentik PRD avec diagramme de séquence Mermaid | Formaliser un cas d'utilisation produit (description + acteurs typés + intent + séquence numérotée + sequenceDiagram), sortie en français, validation interactive |
| `bmad-customize-skills` | Désactiver / réactiver sélectivement les skills BMad-Method par projet (preset Product-only ou sélection custom). Patche aussi `bmad-help.csv` pour neutraliser les required gates | Adapter une install BMad à un projet doc-only ou non-coding : retirer les skills dev/architecture/sprint sans casser BMad. Réversible et ré-applicable après chaque `npx bmad-method install` |
| `use-case-prioritization` | Scorer, prioriser et chiffrer des use cases d'AI/automatisation (BABOK + UiPath Suitability **1-3 avec labels qualitatifs** + Run cost **benchmarké par recherche web** + **confiance hybride** complétude × validation LLM + **synthèse avec jugement LLM** bundling et recommandations stratégiques). Sortie : dossier projet par run avec CSV 38 cols v5, synthèse lisible, et sources auditables | Décider quel use case scoper en premier après un atelier de découverte (post-its, transcripts, inventaires 500+ lignes), construire une enveloppe budgétaire ou faire le triage d'une pipeline d'automatisation. Run cost ancré dans la réalité économique 2026, confiance qui distingue chiffres validés vs estimations, synthèse qui combine math rigoureuse + jugement LLM contextuel |
| `use-case-value` | Priorise les use cases d'AI/automatisation par **impact business chiffré documenté** uniquement (pas d'effort technique). v1.0 : 25 colonnes focus VALEUR, **six sources d'Impact $ explicites** (temps perdu, opportunités manquées, coût des erreurs, saturation main-d'œuvre, dépendances externes, frictions inter-départementales) + pain points + personnes affectées + transversalité. **Règle d'or stricte** : chiffres durs uniquement dans les cellules, **estimations LLM exclusivement en Notes** comme questions à poser au sponsor. Synthèse one-pager en prose narrative française Québec | Produire une note de cadrage business lisible pour un directeur de compte après un atelier de découverte, sans gonfler les chiffres avec des estimations LLM. Complémentaire à `use-case-prioritization` v2.2 : ce skill se concentre sur l'impact, l'effort sera traité par un futur skill séparé qui se branchera sur le CSV de sortie pour fermer la lecture ROI |

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
