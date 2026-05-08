---
name: skill-creator
description: Crée interactivement un nouveau skill Claude Code dans le mono-repo personnel de l'utilisateur (typiquement <github-user>/skills). Détecte ou demande la config (GitHub user, author name, repo name, path local) à la première utilisation et la persiste. Pose des questions sur le slug, la description et les étapes du workflow, génère plugin.json + SKILL.md respectant le format plugin marketplace + standard agentskills.io, met à jour marketplace.json + README.md du repo, et propose de commit + push. À invoquer quand l'utilisateur veut ajouter un nouveau skill à sa marketplace personnelle.
---

# skill-creator — Créer un nouveau skill packagé

Aide n'importe quel utilisateur (peu importe son compte GitHub, son nom, ou ses chemins) à créer un nouveau skill Claude Code dans **son propre** mono-repo de skills. La première fois, le skill détecte ou demande sa config et la sauvegarde. Ensuite chaque création est rapide.

**Réponds dans la langue de l'utilisateur (français par défaut si ambigu). Sois concis. Confirme chaque action en une phrase.**

---

## Étape 0 — Charger ou initialiser la configuration utilisateur

Le skill stocke la config dans `~/.claude/skill-creator/config.json` :

```json
{
  "github_user": "<login GitHub>",
  "author_name": "<nom complet>",
  "author_url": "https://github.com/<login>",
  "repo_name": "<nom du repo, ex: skills>",
  "repo_dir": "<chemin local absolu, ex: /Users/foo/code/skills>",
  "repo_url": "https://github.com/<login>/<repo_name>.git"
}
```

**Logique** :

1. Si `~/.claude/skill-creator/config.json` existe et est valide → charge-le, **affiche un résumé** ("Config: github_user=<X>, repo=<Y>, dir=<Z>") et passe à l'Étape 1.
2. Sinon → autodétecte les défauts puis demande confirmation/édition à l'utilisateur :
   - `github_user` ← `gh api user --jq .login`
   - `author_name` ← `git config --global user.name` (ou `gh api user --jq .name` en fallback)
   - `author_url` ← `https://github.com/<github_user>`
   - `repo_name` ← suggérer `"skills"` (peut être autre chose, ex: `"my-claude-skills"`)
   - `repo_dir` ← suggérer `$HOME/code/<repo_name>`
   - `repo_url` ← `https://github.com/<github_user>/<repo_name>.git`

   Pose ces choix via `AskUserQuestion` (un par question, avec valeur détectée comme première option "Recommandée"). Permet override par "Other".

3. Une fois validée, écris la config dans `~/.claude/skill-creator/config.json` (`mkdir -p ~/.claude/skill-creator` d'abord). Confirme : "Config sauvegardée. Tu peux la modifier à tout moment dans ce fichier."

**Tous les Étapes suivantes utilisent ces variables. JAMAIS de chemin ou nom hardcodé.**

---

## Étape 1 — Vérifications préalables

Lance en parallèle :
- `gh auth status` — l'utilisateur doit être authentifié
- Vérifier que `gh api user --jq .login` retourne bien `<github_user>` de la config (sinon, mismatch — demander quoi faire)
- `git config --global user.email` — doit retourner une valeur

Si quelque chose manque, dis exactement ce qui manque et comment réparer (`gh auth login`, etc.), puis arrête.

---

## Étape 2 — État du repo skills personnel

Vérifie `<repo_dir>` :

- **Cas A : `<repo_dir>` n'existe pas** :
   - Vérifie si le repo GitHub existe : `gh repo view <github_user>/<repo_name> --json name 2>/dev/null`
   - **Cas A.1 : repo GitHub existe** → `git clone <repo_url> <repo_dir>`
   - **Cas A.2 : repo GitHub n'existe pas** → bootstrap un nouveau repo (Étape 2bis)

- **Cas B : `<repo_dir>` existe** → `git -C <repo_dir> pull --ff-only`

Si le pull/clone échoue (conflits, divergence, permissions), dis exactement l'erreur et arrête — l'utilisateur doit régler manuellement.

---

## Étape 2bis — Bootstrap d'un nouveau repo skills (première utilisation)

Si le repo `<github_user>/<repo_name>` n'existe pas encore (sur GitHub ni en local) :

1. Confirme avec `AskUserQuestion` : "Créer le repo `<github_user>/<repo_name>` (public) maintenant ?"
2. Si oui :
   - `mkdir -p <repo_dir>/.claude-plugin <repo_dir>/plugins`
   - Crée `<repo_dir>/.claude-plugin/marketplace.json` :
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
   - Crée `<repo_dir>/README.md` (catalogue vide pour le moment, voir template Annexe A)
   - Crée `<repo_dir>/LICENSE` (MIT, copyright `<author_name>` + année courante via `date +%Y`)
   - `cd <repo_dir> && git init -b main && git add -A && git commit -m "Initial commit: empty <repo_name> marketplace"`
   - `gh repo create <github_user>/<repo_name> --public --description "Marketplace personnelle de skills Claude Code" --source=. --push`
3. Si non, arrête le workflow (l'utilisateur doit créer son repo manuellement avant de réutiliser le skill).

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

Demande successivement :

1. **Langue** de réponse du skill final (français / anglais / autre)
2. **Style** (concis / détaillé)
3. **Nombre d'étapes** entre 3 et 7
4. **Pour chaque étape** : titre court + description (texte libre, 2-5 lignes) + commandes shell impliquées (optionnel)

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

## Étape 10 — Commit + push ?

`AskUserQuestion` :

- **Oui, commit + push maintenant** :
  ```bash
  cd <repo_dir>
  git add -A
  git commit -m "Add <slug> skill"
  git push
  ```
- **Non, je m'en charge plus tard** : affiche le chemin local et les commandes à exécuter à la main.

---

## Étape 11 — Récap final

Affiche :

```
✓ Skill <slug> créé.

Fichiers :
  <repo_dir>/plugins/<slug>/
  ├── .claude-plugin/plugin.json
  └── skills/<slug>/SKILL.md

[Si pushé]
GitHub : https://github.com/<github_user>/<repo_name>/tree/main/plugins/<slug>

Pour l'installer dans Claude Code :
  /plugin marketplace update <repo_name>
  /plugin install <slug>@<repo_name>

Slash command résultant : /<slug>:<slug>

[Si pas pushé]
À faire plus tard :
  cd <repo_dir> && git add -A && git commit -m "Add <slug> skill" && git push
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
