---
name: skill-creator
description: Meta-skill GÉNÉRIQUE et ADAPTATIF qui aide n'importe quel utilisateur à créer ses propres skills Claude Code dans SA propre marketplace personnelle. Fonctionne en deux modes — "github" (avec compte GitHub + gh CLI, push automatique) ou "local" (sans GitHub, marketplace locale uniquement). Détecte automatiquement le mode disponible, basculer vers local si gh manque ou pas authentifié. Persiste la config dans ~/.claude/skill-creator/config.json. Génère des skills eux aussi adaptatifs. À invoquer quand l'utilisateur veut créer un skill, qu'il ait GitHub ou pas.
---

# skill-creator — Créer un nouveau skill packagé (générique, adaptatif)

## Philosophie & use cases

Ce skill est **générique** : il ne suppose RIEN sur l'identité de l'utilisateur courant. Il peut être installé et utilisé par n'importe qui (Samuel, ses amis, son équipe, un random sur Internet) et fonctionnera immédiatement avec **leurs** infos GitHub et **leurs** chemins.

**Use cases concrets** :

1. **Première utilisation par un nouvel utilisateur GitHub** : Alice clone la marketplace `RunLittleTurtle/skills`, installe `skill-creator`, l'invoque. Le skill détecte qu'elle n'a pas encore de config, autodétecte ses infos via `gh api user` et `git config`, lui propose `alice/skills` comme repo cible, et crée le repo GitHub si absent. Tout son setup en 30 secondes.

2. **Utilisateur avec marketplace existante** : Bob a déjà `bob/my-skills` sur GitHub. Il invoque skill-creator, il pointe sa config sur ce repo, et chaque nouveau skill y est ajouté.

3. **Utilisateur SANS GitHub (mode local)** : Charlie n'a pas de compte GitHub, pas de `gh` CLI, ou ne veut pas publier. Le skill détecte l'absence de `gh` et bascule en **mode local** : tout est généré dans `~/code/skills/` (ou autre dossier choisi), Git local optionnel, pas de push. Pour utiliser ses skills, il fait `/plugin marketplace add ~/code/skills` dans Claude Code (chemin local). Aucune dépendance à Internet ou à un compte.

4. **Multi-utilisateurs / multi-machines** : Diane travaille sur 2 Mac (perso et boulot). Sur chaque Mac, la première utilisation initialise la config locale dans `~/.claude/skill-creator/config.json`. Sa marketplace cible reste la même (mode github avec `diane/skills`, ou mode local avec sync iCloud entre les 2 Macs).

5. **Skills générés aussi adaptatifs** : skill-creator pousse l'utilisateur à rendre ses propres skills génériques (pas de chemins perso hardcodés, pas de noms d'utilisateur figés, utilisation de `gh api` ou `git config` ou `$HOME` pour détection). Ainsi les skills créés sont eux-mêmes partageables.

**Ce que ce skill ne fait PAS** :
- Il ne suppose pas que l'utilisateur s'appelle "Samuel" ou utilise `RunLittleTurtle` comme login.
- Il ne suppose pas que l'utilisateur a un compte GitHub.
- Il n'écrit jamais de chemin absolu dans les skills générés (ex: `/Users/samuel/...`).
- Il n'utilise jamais le repo de quelqu'un d'autre comme cible (chacun pousse dans le sien, ou reste local).
- Il ne demande jamais à l'utilisateur d'installer `gh` s'il ne veut pas — le mode local est un citoyen de première classe, pas un fallback dégradé.

---

**Réponds dans la langue de l'utilisateur (français par défaut si ambigu). Sois concis. Confirme chaque action en une phrase.**

---

## Étape 0 — Charger ou initialiser la configuration utilisateur

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

## Étape 7 — Construire les fichiers du nouveau skill

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
  "homepage": "https://github.com/<github_user>/<repo_name>"
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

---

## Étape 8 — Mettre à jour `marketplace.json`

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

## Étape 11 — Récap final

### Si `mode: "github"` et pushé
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
