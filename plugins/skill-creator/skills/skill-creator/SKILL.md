---
name: skill-creator
description: Meta-skill GÉNÉRIQUE et ADAPTATIF qui aide n'importe quel utilisateur à créer un skill Claude Code (ou compatible agentskills.io). Supporte 3 cibles selon ce que l'utilisateur veut faire — (A) marketplace personnelle multi-skills sur GitHub ou en local, (B) skill standalone perso direct dans ~/.claude/skills/ sans marketplace, (C) skill pour un autre outil (OpenCode, GitHub Copilot Enterprise, Cursor, Claude Desktop, etc.). Détecte automatiquement les outils disponibles, demande la cible appropriée, et adapte tout le workflow. Persiste la config dans ~/.claude/skill-creator/config.json. À invoquer quand l'utilisateur veut créer un skill, peu importe son setup ou ses intentions.
---

# skill-creator — Créer un nouveau skill packagé (générique, adaptatif)

## Philosophie & use cases

Ce skill est **générique** : il ne suppose RIEN sur l'identité de l'utilisateur courant. Il peut être installé et utilisé par n'importe qui (Samuel, ses amis, son équipe, un random sur Internet) et fonctionnera immédiatement avec **leurs** infos GitHub et **leurs** chemins.

**Use cases concrets** (3 grandes catégories — A/B/C — choisies à l'Étape 0.0) :

### Type A — Marketplace personnelle (multi-skills, partageable)

1. **Nouvel utilisateur GitHub** : Alice clone `RunLittleTurtle/skills`, installe skill-creator, l'invoque. Autodétection via `gh api user`, propose `alice/skills`, crée le repo GitHub si absent. Setup en 30s.

2. **Utilisateur avec marketplace existante** : Bob a déjà `bob/my-skills`. La config pointe dessus, chaque nouveau skill y est ajouté + commit + push.

3. **Marketplace LOCALE (sans GitHub)** : Charlie n'a pas de `gh` ou ne veut pas publier. Tout dans `~/code/skills/`, install via `/plugin marketplace add ~/code/skills`. Git local optionnel. Aucune dépendance Internet.

### Type B — Skill standalone perso (juste pour soi, sans marketplace)

4. **Skill perso rapide** : Diane veut juste un skill `/check-pr` dans son Claude Code, pas besoin de le partager ou versionner. skill-creator génère directement `~/.claude/skills/check-pr/SKILL.md` — un seul fichier, pas de marketplace, pas de plugin.json. Disponible immédiatement.

### Type C — Skill pour un autre outil (OpenCode, Copilot, Cursor, etc.)

5. **Copilot Enterprise** : Eve travaille avec GitHub Copilot Enterprise au boulot. Elle veut créer un skill pour son équipe. skill-creator génère un `SKILL.md` au standard agentskills.io et lui dit où le placer dans son outil (selon la doc Copilot). Pas de plugin.json Claude Code, pas de marketplace.

6. **OpenCode / Cursor / Claude Desktop** : pareil — skill généré au standard ouvert, dossier cible adapté à l'outil.

### Use case transversal — Skills générés adaptatifs

7. **Skills eux-mêmes adaptatifs** : skill-creator pousse l'utilisateur à rendre ses propres skills génériques (pas de chemins perso hardcodés, pas de noms d'utilisateur figés, utilisation de `gh api`, `git config`, `$HOME` pour détection). Les skills créés sont alors partageables avec d'autres.

**Ce que ce skill ne fait PAS** :
- Il ne suppose pas l'identité de l'utilisateur (login, nom, organisation).
- Il ne suppose pas que l'utilisateur a un compte GitHub.
- Il ne suppose pas que l'utilisateur veut une marketplace (Type B existe).
- Il ne suppose pas que l'utilisateur veut Claude Code (Type C existe pour autres outils).
- Il n'écrit jamais de chemin absolu hardcodé dans les skills générés (ex: `/Users/samuel/...`).
- Il n'utilise jamais le repo de quelqu'un d'autre comme cible.
- Il ne demande jamais à installer `gh` ou créer un compte GitHub si l'utilisateur ne veut pas.

---

**Réponds dans la langue de l'utilisateur (français par défaut si ambigu). Sois concis. Confirme chaque action en une phrase.**

---

## Étape 0.0 — Type de skill à créer (BRANCHEMENT PRINCIPAL)

**Avant tout**, demande à l'utilisateur via `AskUserQuestion` :

> "Que veux-tu faire avec ce nouveau skill ?
>
> - **A) Ajouter à ma marketplace personnelle** (recommandé pour skills partageables) : structure complète plugin marketplace, multi-skills dans un seul dépôt (`<user>/skills` ou dossier local), partage facile via `/plugin marketplace add`.
>
> - **B) Skill perso standalone** (juste pour moi, ici, maintenant) : un seul fichier `~/.claude/skills/<slug>/SKILL.md`, pas de marketplace, pas de Git, pas de partage. Utilisable immédiatement dans Claude Code.
>
> - **C) Skill pour un autre outil** (OpenCode, GitHub Copilot, Cursor, Claude Desktop, etc.) : SKILL.md au standard agentskills.io placé dans le dossier skills de l'outil cible. Tu m'indiques l'outil ou le chemin."

**Selon la réponse** :
- **Type A** → continuer à l'Étape 0.1 (config marketplace).
- **Type B** → SKIP Étapes 0.1, 1, 2, 2bis, 8, 9, 10. Aller direct à l'Étape 3 (slug). À l'Étape 7, écris UNIQUEMENT `~/.claude/skills/<slug>/SKILL.md`. Pas de plugin.json, pas de Git.
- **Type C** → SKIP les mêmes étapes. À l'Étape 7, écris UNIQUEMENT le SKILL.md dans le dossier cible (voir Étape 0.5 pour déterminer le dossier).

---

## Étape 0.5 — Dossier cible (Type C uniquement)

Si Type C, demande via `AskUserQuestion` :

> "Quel outil cible utilises-tu ? (le skill sera placé dans son dossier skills)"

Options proposées (avec dossier par défaut) :
- **OpenCode** → `~/.opencode/skills/`
- **GitHub Copilot** → variable selon config (demande à l'utilisateur ou voir [docs Copilot](https://docs.github.com/en/copilot/concepts/agents/about-agent-skills))
- **Cursor** → variable (voir [docs Cursor](https://cursor.com/docs/context/skills))
- **Claude Desktop (macOS)** → `~/Library/Application Support/Claude/skills/`
- **Autre / chemin personnalisé** → l'utilisateur saisit le chemin via "Other"

Vérifie que le dossier existe (sinon, demande de le créer ou de corriger). Stocke dans une variable locale `<target_dir>` pour cette session — pas dans la config persistée (chaque skill peut viser un outil différent).

---

## Étape 0 — Charger ou initialiser la configuration utilisateur (Type A uniquement)

Le skill stocke la config dans `~/.claude/skill-creator/config.json`. Format :

```json
{
  "mode": "github" | "local",
  "github_user": "<login GitHub, omis en mode local>",
  "author_name": "<nom complet>",
  "author_url": "<URL profile GitHub si mode github, sinon omis>",
  "repo_name": "<nom du repo/dossier, ex: skills>",
  "repo_dir": "<chemin local absolu, ex: /Users/foo/code/skills>",
  "repo_url": "<URL git si mode github, omis en mode local>"
}
```

**Logique** :

### 0.1 — Si `~/.claude/skill-creator/config.json` existe et est valide
Charge-le, affiche un résumé ("Mode: <github|local>, repo: <Y>, dir: <Z>") et passe à l'Étape 1.

### 0.2 — Sinon, détecter le mode disponible

En parallèle :
- `command -v gh` (existe ?)
- Si oui : `gh auth status 2>&1` (authentifié ?)

**Cas A — `gh` absent OU pas authentifié** :
Demande via `AskUserQuestion` :
> "Tu n'as pas (ou pas configuré) `gh` CLI. Tu veux :
> - **Mode local** (recommandé pour toi) : créer tes skills sans GitHub, marketplace locale uniquement
> - **Configurer GitHub d'abord** : je m'arrête, tu lances `gh auth login` puis tu me ré-invoques
> - **Autre** : précise"

Si choix "local" → mettre `mode: "local"` dans la config.

**Cas B — `gh` présent et authentifié** :
Demande via `AskUserQuestion` :
> "Mode pour ta marketplace de skills :
> - **GitHub** (recommandé) : repo public/privé sur GitHub, push automatique, partageable par lien
> - **Local** : fichiers locaux uniquement, pas de push, marketplace via chemin local"

### 0.3 — Auto-détecter les autres champs selon le mode

**Si `mode: "github"`** :
- `github_user` ← `gh api user --jq .login`
- `author_name` ← `gh api user --jq .name` (ou `git config --global user.name` si null)
- `author_url` ← `https://github.com/<github_user>`
- `repo_name` ← suggérer `"skills"`
- `repo_dir` ← suggérer `$HOME/code/<repo_name>`
- `repo_url` ← `https://github.com/<github_user>/<repo_name>.git`

**Si `mode: "local"`** :
- `author_name` ← `git config --global user.name` (si vide, demander)
- `repo_name` ← suggérer `"skills"`
- `repo_dir` ← suggérer `$HOME/code/<repo_name>`
- (pas de `github_user`, `author_url`, `repo_url`)

Pose ces choix via `AskUserQuestion` (valeur détectée comme première option "Recommandée"). Permet override.

### 0.4 — Sauvegarder
Écris la config dans `~/.claude/skill-creator/config.json` (`mkdir -p ~/.claude/skill-creator` d'abord). Confirme : "Config sauvegardée (mode: <mode>). Tu peux la modifier à tout moment dans ce fichier."

**Tous les Étapes suivantes utilisent ces variables. JAMAIS de chemin ou nom hardcodé.**

---

## Étape 1 — Vérifications préalables (selon mode)

### Si `mode: "github"`
Lance en parallèle :
- `gh auth status` — l'utilisateur doit être authentifié
- Vérifier que `gh api user --jq .login` retourne bien `<github_user>` de la config (sinon, mismatch — demander quoi faire : update config ? changer de compte ?)
- `git config --global user.email` — doit retourner une valeur

Si quelque chose manque, dis exactement ce qui manque et propose des solutions (réparer, ou basculer en mode local), puis arrête ou continue selon la réponse.

### Si `mode: "local"`
Lance :
- `command -v git` — git doit être installé (sinon, dis-le et arrête : sans git on ne peut pas init un repo, mais on peut quand même créer les fichiers — demande à l'utilisateur s'il veut continuer SANS git)
- `git config --global user.name` (informatif, optionnel en mode local)

Pas besoin de `gh` ni de connexion Internet en mode local.

---

## Étape 2 — État du repo skills personnel (selon mode)

### Si `mode: "github"`
Vérifie `<repo_dir>` :

- **Cas A : `<repo_dir>` n'existe pas** :
   - Vérifie si le repo GitHub existe : `gh repo view <github_user>/<repo_name> --json name 2>/dev/null`
   - **Cas A.1 : repo GitHub existe** → `git clone <repo_url> <repo_dir>`
   - **Cas A.2 : repo GitHub n'existe pas** → bootstrap un nouveau repo (Étape 2bis)

- **Cas B : `<repo_dir>` existe** → `git -C <repo_dir> pull --ff-only`

Si le pull/clone échoue (conflits, divergence, permissions), dis exactement l'erreur et arrête.

### Si `mode: "local"`
Vérifie `<repo_dir>` :

- **Cas A : `<repo_dir>` n'existe pas** → bootstrap (Étape 2bis local).
- **Cas B : `<repo_dir>` existe avec `.claude-plugin/marketplace.json`** → utilise tel quel, pas besoin de pull.
- **Cas C : `<repo_dir>` existe mais pas de `marketplace.json`** → demande à l'utilisateur : "Le dossier existe mais ne ressemble pas à une marketplace skills. Continuer en l'utilisant quand même (création de la structure) ou choisir un autre dossier ?"

---

## Étape 2bis — Bootstrap d'un nouveau repo skills (première utilisation)

### Si `mode: "github"`

1. Confirme via `AskUserQuestion` : "Créer le repo `<github_user>/<repo_name>` (public) maintenant ?"
2. Si oui :
   - `mkdir -p <repo_dir>/.claude-plugin <repo_dir>/plugins`
   - Crée `<repo_dir>/.claude-plugin/marketplace.json`, `README.md`, `LICENSE` (templates ci-dessous)
   - `cd <repo_dir> && git init -b main && git add -A && git commit -m "Initial commit: empty <repo_name> marketplace"`
   - `gh repo create <github_user>/<repo_name> --public --description "Marketplace personnelle de skills Claude Code" --source=. --push`
3. Si non, arrête le workflow.

### Si `mode: "local"`

1. Confirme via `AskUserQuestion` : "Créer le dossier `<repo_dir>` (marketplace locale) maintenant ?"
2. Si oui :
   - `mkdir -p <repo_dir>/.claude-plugin <repo_dir>/plugins`
   - Crée `<repo_dir>/.claude-plugin/marketplace.json`, `README.md`, `LICENSE` (templates adaptés ci-dessous)
   - Optionnel (demande à l'utilisateur) : `cd <repo_dir> && git init -b main && git add -A && git commit -m "Initial commit"` (utile pour versioning local même sans GitHub)
3. Si non, arrête.

### Templates pour le bootstrap

**`marketplace.json` (mode github)** :
```json
{
  "name": "<repo_name>",
  "description": "Marketplace personnelle de skills Claude Code de <author_name>",
  "owner": {
    "name": "<author_name>",
    "url": "<author_url>"
  },
  "plugins": []
}
```

**`marketplace.json` (mode local)** : identique mais SANS `owner.url`.

**`LICENSE`** : MIT classique avec `Copyright (c) <année courante> <author_name>`.

**`README.md`** : template Annexe A adapté au mode (instructions install différentes).

---

## Étape 3 — Demander le slug du nouveau skill

`AskUserQuestion` avec une question texte libre (option "Other") :

> "Quel est le slug du nouveau skill ? Format kebab-case, ex: `pr-review`, `weekly-recap`, `daily-standup`."

**Validation** :
- Pattern `^[a-z][a-z0-9-]*$` (lowercase, chiffres et tirets, commence par une lettre)
- `<repo_dir>/plugins/<slug>/` ne doit pas exister

Si invalide ou déjà existant, explique pourquoi et redemande.

---

## Étape 4 — Demander la description (frontmatter)

`AskUserQuestion` avec texte libre :

> "Décris en 1-3 phrases QUAND ce skill doit être invoqué. Commence par un verbe d'action (Coordonner..., Générer..., Analyser...). Cette description sera lue par le LLM pour décider d'activer le skill."

Garde le texte tel quel (pas de troncature, pas de reformulation auto).

---

## Étape 5 — Mode de rédaction du body

`AskUserQuestion` avec 3 options :
- **Interactif** : poser des questions sur les étapes, je génère tout
- **Coller un brouillon** : l'utilisateur fournit le markdown complet
- **Squelette minimal** : générer un template à remplir plus tard

---

## Étape 6a — Mode interactif

**Avant de commencer la rédaction**, rappelle à l'utilisateur le **principe d'adaptabilité** :

> "Ce skill que tu crées sera potentiellement installé chez d'autres. Évite de hardcoder ton nom, ton login GitHub, des chemins absolus comme `/Users/<toi>/...`, ou des URLs spécifiques à ton compte. Préfère des détections runtime (`gh api user --jq .login`, `git config user.name`, `$HOME`, etc.) ou des variables config. Si le skill DOIT être perso (ex: workflow lié à ton équipe), dis-le explicitement dans la description."

Puis demande successivement :

1. **Langue** de réponse du skill final (français / anglais / autre)
2. **Style** (concis / détaillé)
3. **Portée** : skill **générique** (utilisable par d'autres) ou **personnel** (lié à TON setup spécifique) ? Si personnel, le mentionner clairement dans la description du frontmatter.
4. **Nombre d'étapes** entre 3 et 7
5. **Pour chaque étape** : titre court + description (texte libre, 2-5 lignes) + commandes shell impliquées (optionnel)

Pendant la rédaction, si tu détectes des hardcoded paths/noms dans la description d'une étape, alerte l'utilisateur : "Tu mentionnes `<X>` qui semble spécifique à ton setup. Veux-tu le rendre dynamique (détection runtime) ?".

Construis le body progressivement au format markdown :

```markdown
# <Nom lisible du skill>

<Phrase d'accroche reprenant la description>.

**<Instructions de style : langue + concision>**

## Étape 1 — <Titre>

<Description>

[Si commandes : bloc de code shell]

## Étape 2 — <Titre>

...
```

---

## Étape 6b — Mode "coller un brouillon"

`AskUserQuestion` avec texte libre :

> "Colle ici le contenu markdown complet du body de ton skill (sans le frontmatter `---`, juste le contenu après)."

Utilise tel quel.

---

## Étape 6c — Mode "squelette minimal"

Génère :

```markdown
# <Nom du skill>

<description telle que fournie à l'étape 4>

**Réponds en français. Sois concis.**

## Étape 1 — TODO

À remplir.

## Étape 2 — TODO

À remplir.

## Étape 3 — TODO

À remplir.
```

---

## Étape 7 — Construire les fichiers du nouveau skill (selon Type)

### Type A — Marketplace personnelle

Crée la structure dans `<repo_dir>` :

```
<repo_dir>/plugins/<slug>/
├── .claude-plugin/plugin.json
└── skills/<slug>/SKILL.md
```

**`<repo_dir>/plugins/<slug>/.claude-plugin/plugin.json`** :
```json
{
  "name": "<slug>",
  "description": "<description complète frontmatter>",
  "version": "1.0.0",
  "author": {
    "name": "<author_name>",
    "url": "<author_url>"
  },
  "license": "MIT",
  "homepage": "<https://github.com/<github_user>/<repo_name>> ou omis en mode local"
}
```

**`<repo_dir>/plugins/<slug>/skills/<slug>/SKILL.md`** :
```markdown
---
name: <slug>
description: <description>
---

<body construit à l'étape 6>
```

### Type B — Skill standalone perso

Crée UNIQUEMENT :
```
~/.claude/skills/<slug>/SKILL.md
```
Pas de plugin.json, pas de marketplace.json. Le SKILL.md a juste le frontmatter `name` + `description` + body. Disponible immédiatement dans Claude Code (slash command `/<slug>`).

`mkdir -p ~/.claude/skills/<slug>` avant d'écrire.

### Type C — Skill pour un autre outil

Crée UNIQUEMENT :
```
<target_dir>/<slug>/SKILL.md
```

(`<target_dir>` vient de l'Étape 0.5). Pas de plugin.json (les autres outils ne s'en servent pas). Le SKILL.md respecte le standard agentskills.io.

`mkdir -p <target_dir>/<slug>` avant d'écrire.

---

**Pour Types B et C : SKIP les Étapes 8, 9, 10. Va directement à l'Étape 11 (récap final adapté).**

---

## Étape 8 — Mettre à jour `marketplace.json` (Type A uniquement)

Lis `<repo_dir>/.claude-plugin/marketplace.json` (JSON valide). Ajoute une nouvelle entrée à la fin du tableau `plugins` :

```json
{
  "name": "<slug>",
  "source": "./plugins/<slug>",
  "description": "<description courte, 1 phrase>"
}
```

**Format `source` important** : utilise toujours `"./plugins/<slug>"` (chemin relatif explicite). Le format court `"<slug>"` avec `metadata.pluginRoot` est rejeté par le validator actuel.

Préserve le formatting (2 espaces d'indentation, virgules correctes entre entrées). Réécris le fichier complet.

**Validation** : après écriture, lance `claude plugin validate <repo_dir>` et résous toute erreur avant de continuer.

---

## Étape 9 — Mettre à jour `README.md`

Lis `<repo_dir>/README.md`. Trouve la section `## Skills disponibles` et son tableau. Ajoute une nouvelle ligne :

```
| `<slug>` | <description courte 1 phrase> | <use case 1 phrase> |
```

Si la section ou le tableau n'existent pas (par ex. README édité manuellement), dis-le à l'utilisateur et demande comment procéder (créer la section, ignorer, autre).

---

## Étape 10 — Commit (et push si applicable) ?

### Si `mode: "github"`
`AskUserQuestion` :
- **Oui, commit + push maintenant** :
  ```bash
  cd <repo_dir> && git add -A && git commit -m "Add <slug> skill" && git push
  ```
- **Non, plus tard** : affiche les commandes manuelles.

### Si `mode: "local"`
`AskUserQuestion` :
- **Oui, commit local** (si le repo a un `.git`) :
  ```bash
  cd <repo_dir> && git add -A && git commit -m "Add <slug> skill"
  ```
- **Non, juste laisser les fichiers** : pas de commit, l'utilisateur gère git lui-même.

Aucun push possible en mode local — c'est par design, pas un bug.

---

## Étape 11 — Récap final (selon Type)

### Type B — Skill standalone perso
```
✓ Skill <slug> créé localement.

Fichier : ~/.claude/skills/<slug>/SKILL.md

Disponible immédiatement dans Claude Code via : /<slug>

Pour le supprimer : rm -rf ~/.claude/skills/<slug>
Pour l'éditer : édite directement ~/.claude/skills/<slug>/SKILL.md
```

### Type C — Skill pour un autre outil
```
✓ Skill <slug> créé pour <outil>.

Fichier : <target_dir>/<slug>/SKILL.md

Format : standard agentskills.io (frontmatter name + description, body markdown).

Pour activer : consulte la doc de <outil> (typiquement, le skill est détecté
automatiquement au prochain démarrage de l'outil).

Pour partager : envoie le dossier <target_dir>/<slug>/ ou le contenu du SKILL.md.
```

### Type A — Marketplace personnelle, mode `"github"` et pushé
```
✓ Skill <slug> créé et publié.

Fichiers : <repo_dir>/plugins/<slug>/
GitHub : https://github.com/<github_user>/<repo_name>/tree/main/plugins/<slug>

Pour l'installer dans Claude Code (sur ta machine ou celle d'un ami) :
  /plugin marketplace add <github_user>/<repo_name>     (si pas déjà ajoutée)
  /plugin marketplace update <repo_name>
  /plugin install <slug>@<repo_name>

Slash command : /<slug>:<slug>
```

### Si `mode: "github"` et pas pushé
```
✓ Skill <slug> créé localement.

Fichiers : <repo_dir>/plugins/<slug>/

À faire plus tard pour publier :
  cd <repo_dir> && git add -A && git commit -m "Add <slug> skill" && git push
```

### Si `mode: "local"`
```
✓ Skill <slug> créé localement (mode local, pas de GitHub).

Fichiers : <repo_dir>/plugins/<slug>/

Pour l'installer dans Claude Code :
  /plugin marketplace add <repo_dir>           (chemin local absolu, pas owner/repo)
  /plugin marketplace update <repo_name>
  /plugin install <slug>@<repo_name>

Slash command : /<slug>:<slug>

Pour partager avec un ami : zippe le dossier <repo_dir>/ et envoie-lui (il fait
/plugin marketplace add <chemin du dossier dézippé>). Ou bien passe en mode
github via /skill-creator pour publier.
```

---

## Annexe A — Template README pour bootstrap

Quand on bootstrap un nouveau repo (Étape 2bis), voici le template à utiliser pour `<repo_dir>/README.md` (remplacer les `<vars>` par les valeurs de la config) :

```markdown
# <repo_name> — Marketplace de skills de <author_name>

Marketplace Claude Code regroupant les skills publics de <author_name>. Une seule commande pour les avoir tous, tu actives ceux que tu veux.

## Installation

### Pour Claude Code

​```
/plugin marketplace add <github_user>/<repo_name>
​```

Va dans l'onglet **Discover** et active les skills voulus (Espace pour toggle), ou installe en CLI :

​```
/plugin install <nom-du-skill>@<repo_name>
​```

- **Mise à jour** : `/plugin marketplace update <repo_name>`
- **Désinstallation** : `/plugin uninstall <nom>@<repo_name>`

### Pour les autres outils (Claude Desktop, OpenCode, GitHub Copilot, Cursor, etc.)

Les skills respectent le standard ouvert [agentskills.io](https://agentskills.io). Clone le repo et copie le skill voulu dans le dossier skills de ton outil :

​```bash
git clone https://github.com/<github_user>/<repo_name>.git
cp -R <repo_name>/plugins/<nom-du-skill>/skills/<nom-du-skill> <DOSSIER_SKILLS_DE_TON_OUTIL>/
​```

## Skills disponibles

| Nom | Description | Use case |
|---|---|---|
| _(aucun pour le moment — ajoute-en avec `skill-creator`)_ | | |

## Licence

MIT — voir [LICENSE](./LICENSE).
```

---

## Notes importantes pour Claude pendant l'exécution

- **Ne jamais hardcoder de chemin ou de nom**. Tout passe par les variables de la config (Étape 0).
- **Ne crée jamais de symlink**. Le skill vit uniquement dans `<repo_dir>/plugins/<slug>/`. L'install côté utilisateur passe par `/plugin install`, pas par un symlink manuel.
- **Ne crée pas de nouveau repo GitHub par skill**. Tout va dans le mono-repo `<github_user>/<repo_name>`. Pas de `gh repo create` sauf en bootstrap (Étape 2bis).
- **Ne modifie jamais les autres skills existants**. Touche uniquement aux fichiers du nouveau skill + `marketplace.json` + `README.md`.
- **Si une étape échoue** (pull conflict, JSON invalide, slug en doublon, push permission), arrête et dis exactement ce qui s'est passé. Ne tente pas de réparer en silence.
- **Frontmatter SKILL.md** : exactement `name` + `description` (standard agentskills.io). Pas de `version`, `tags` ou autres champs non standard.
- **Mismatch GitHub user** : si `gh api user --jq .login` retourne un login différent de celui dans la config, prévenir l'utilisateur avant de continuer (peut-être qu'il a changé de compte ou que la config est obsolète).
