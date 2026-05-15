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

### Flow d'usage — Business Analysis → Produit → Directeur de compte

```mermaid
flowchart LR
  A[use-case-value<br/><i>Business Analysis</i>] --> B[use-case-prioritization<br/><i>draft</i>]
  B --> C[scenario-uc<br/><i>Product Management</i>]
  C --> C2[scenario-uc-v2<br/><i>version parallèle</i>]
  C --> D[product-brief<br/><i>Product Management</i>]
  C --> E[mermaid-flow<br/><i>Directeur de compte</i>]
```

- **use-case-value** : analyse d'impact business chiffré (chiffres durs uniquement, root cause avant chiffrage).
- **use-case-prioritization** *(draft)* : ajoute effort, ROI, Run cost benchmarké.
- **scenario-uc** : formalise le use case retenu en scénario PRD.
- **scenario-uc-v2** : version parallèle de `scenario-uc` qui ajoute scénarios alternatifs HEC (suffixes `a/b/c`), boucles `LOOP / FIN LOOP` (alignées sur `loop ... end` Mermaid), préfixe `AS-IS_v<N>` / `TO-BE_v<N>` dans le H1, et validation interactive renforcée.
- **product-brief** : transforme inputs hétérogènes (BA, transcripts, OKRs) en product brief one-pager au format PRD Authentik. v2 — posture Product Manager Senior, scan préliminaire de la doc, validation par section via AskUserQuestion, citations verbatim complètes, AARRR conditionnel. Version 1 archivée sous `product-brief-v1`.
- **mermaid-flow** : vulgarise les scénarios pour un Directeur de compte ou stakeholder non technique.

Les autres skills (`coordination`, `agent-talk`, `skill-creator`, `bmad-customize-skills`) sont indépendants de ce flow.

### Tableau récapitulatif

| Nom | Description |
|---|---|
| `use-case-value` | Priorise les use cases d'AI/automatisation par impact business chiffré (6 sources d'impact $, root cause avant chiffrage, règle d'or v1.3 avec inférences sur paire d'anchors verbaux et défauts conservateurs, Score amplifié pour transverse via Personnes_factor paliers + Transversalité_factor, synthèse Top 10 × 3 lignes décisive). |
| `use-case-prioritization` `[draft]` | Score + priorise + chiffre les use cases (BABOK + UiPath Suitability + Run cost benchmarké web + confiance hybride). |
| `scenario-uc` | Transforme tout input (md/PDF/image/idée) en scénario use-case au format PRD avec diagramme de séquence Mermaid. |
| `scenario-uc-v2` | Version parallèle de `scenario-uc` qui ajoute scénarios alternatifs au format HEC (suffixes `1.3a/1.3b`), boucles `LOOP / FIN LOOP` (alignées sur `loop ... end` Mermaid), titre via frontmatter Mermaid, préfixe `AS-IS_v<N>` / `TO-BE_v<N>` dans le H1, et validation interactive renforcée (questions bloquantes sur ambiguïtés). |
| `product-brief` | Transforme inputs hétérogènes (notes BA, data points, transcripts, insights discovery, OKRs) en product brief one-pager au format PRD Authentik. v2 : posture Product Manager Senior, scan préliminaire de la doc, validation par section via AskUserQuestion, citations verbatim complètes (sans coupure), contraintes flexibles au-delà de Budget/Temps/Pourquoi maintenant, OKR avec validation séparée Objectives/KR (max 3 par catégorie) et paragraphe épuré, AARRR conditionnel (alternatives Avant/Après ou ROI pour workflows internes), référencement des scénarios formalisés. |
| `product-brief-v1` `[archivé]` | Version 1 originale du skill product-brief, conservée pour reproductibilité. La version active est `product-brief`. |
| `mermaid-flow` | Transforme un flow en flowchart Mermaid simplifié pour personnes peu techniques (max 10 étapes, emojis acteurs). |
| `bmad-customize-skills` | Désactive/réactive sélectivement les skills BMad-Method par projet (preset Product-only ou sélection custom). |
| `skill-creator` | Meta-skill générique pour créer un nouveau skill dans ta propre marketplace (pas la mienne). |
| `coordination` | Coordonne plusieurs instances Claude qui travaillent en parallèle sur le même repo via locks markdown. |
| `agent-talk` | Permet à deux instances Claude (même repo nested OU cross-projet) d'échanger des messages structurés (conversation, rapport, handoff) via un dossier-bridge partagé et des pings courts. Complète `coordination`. |
| `product-management` | Boîte à outils PM adaptée pour Claude Code (fork du plugin officiel Anthropic) : 8 skills auto-déclenchés (write-spec, roadmap, stakeholder updates, recherche utilisateur, analyse concurrentielle, métriques, sprint planning, brainstorming) + `/brainstorm`. Sans `.mcp.json` embarqué — opt-in MCP via ta config globale, fallback fichiers locaux + paste. |
| `bug-us-mapping` | Croise un export CSV de User Stories Jira avec un export CSV de bugs pour identifier les US Done qui ne sont pas réellement complétées à 100%. v1.1.0 : auto-détection avec validation interactive, mapping sémantique strict (règle anti-forçage), table unique triée par % complétion croissant (30/50/70/85/100), section dédiée 'Bugs non liés à une US' avec validation PO par bug orphelin (laisser / rattacher faible / candidat nouvelle US / ignorer), versioning auto du fichier de sortie. |

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
