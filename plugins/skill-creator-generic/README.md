# skill-creator-generic

> Create a new agent skill OR modify an existing one, in a personal marketplace (Type A, stable or beta), as a personal standalone (Type B), or for another tool compatible with agentskills.io (Type C). Identity-free and runtime-configured. Separates the GitHub repo name from the marketplace identifier so that two users with the same repo name never collide inside Claude Code.

- **Created**: `2026-05-20`
- **Last updated**: `2026-05-20`

```mermaid
flowchart LR
    A[skill idea] --> B[skill-creator-generic]
    B --> C[plugin files + marketplace entry]
```

## Installation

```
/plugin marketplace add RunLittleTurtle/skills
/plugin install skill-creator-generic@skills
```

Slash: `/skill-creator-generic`. First invocation asks for GitHub CLI auth, marketplace names (with separate fields for the repo slug and the Claude Code marketplace identifier), repo visibility, and target tool if not Claude Code.

## Fork notes

This is a fork of [`flo351/skills` → `skill-creator`](https://github.com/flo351/skills/tree/main/plugins/skill-creator) by [@flo351](https://github.com/flo351). The fork adds one structural change: `<context>_marketplace_name` is a config field distinct from `<context>_name`. The repo slug stays short and natural (`skills`), the marketplace identifier defaults to `<github_user>-<repo_name>` (e.g. `alice-skills`) to avoid collisions inside Claude Code when two users own repos with the same name. See the SKILL.md "Why two names" section for the full rationale.

## License

MIT — see [LICENSE](../../LICENSE).
