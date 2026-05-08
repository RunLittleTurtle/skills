---
name: skill-creator
description: Crée interactivement un nouveau skill Claude Code dans le mono-repo RunLittleTurtle/skills (~/code/skills/). Pose des questions sur le slug, la description et les étapes du workflow, génère plugin.json + SKILL.md respectant le format plugin marketplace + standard agentskills.io, met à jour marketplace.json + README.md du repo, et propose de commit + push. À invoquer quand l'utilisateur veut ajouter un nouveau skill à sa marketplace personnelle RunLittleTurtle/skills.
---

# skill-creator — Créer un nouveau skill packagé

Aide l'utilisateur à créer un nouveau skill Claude Code dans le mono-repo `~/code/skills/` (publié sur https://github.com/RunLittleTurtle/skills). Workflow interactif : tu poses les questions, tu génères les fichiers, tu mets à jour marketplace.json + README, et tu proposes de push.

**Réponds en français. Sois concis. Confirme chaque action en une phrase.**

## Étape 1 — Vérifications préalables

Lance en parallèle :
- `gh auth status` — l'utilisateur doit être authentifié sur GitHub
- `git config --global user.name` et `git config --global user.email` — doivent retourner une valeur

Si l'un manque, dis exactement ce qui manque et comment le réparer (`gh auth login` ou `git config --global user.name "..."`), puis arrête.

## Étape 2 — Synchroniser le repo local

Vérifie `~/code/skills/` :
- **Si absent** : `git clone https://github.com/RunLittleTurtle/skills.git ~/code/skills`
- **Si présent** : `git -C ~/code/skills pull --ff-only`

Si le pull échoue (conflits, divergence), dis-le à l'utilisateur et arrête — il doit régler manuellement avant de continuer.

## Étape 3 — Demander le slug du nouveau skill

Utilise `AskUserQuestion` avec une question texte libre (option "Other") :

> "Quel est le slug du nouveau skill ? Format kebab-case, ex: `pr-review`, `weekly-recap`, `daily-standup`."

**Validation** :
- Pattern `^[a-z][a-z0-9-]*$` (lowercase, chiffres et tirets, commence par une lettre)
- `~/code/skills/plugins/<slug>/` ne doit pas exister

Si invalide ou existant, explique et redemande.

## Étape 4 — Demander la description (frontmatter)

`AskUserQuestion` avec texte libre :

> "Décris en 1-3 phrases QUAND ce skill doit être invoqué. Commence par un verbe d'action (Coordonner..., Générer..., Analyser...). Cette description sera lue par le LLM pour décider d'activer le skill."

Garde le texte tel quel (pas de troncature, pas de reformulation auto).

## Étape 5 — Mode de rédaction du body

`AskUserQuestion` avec 3 options :
- **Interactif** : poser des questions sur les étapes, je génère tout
- **Coller un brouillon** : l'utilisateur fournit le markdown complet
- **Squelette minimal** : générer un template à remplir plus tard

## Étape 6a — Mode interactif

Demande successivement (via `AskUserQuestion` texte libre ou choix multiples selon le cas) :

1. **Langue** : français / anglais / autre
2. **Style** : concis / détaillé
3. **Nombre d'étapes** : entre 3 et 7
4. **Pour chaque étape** :
   - Titre court (1 ligne)
   - Description : que fait-on à cette étape (texte libre, 2-5 lignes)
   - Commandes shell impliquées (optionnel)

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

## Étape 6b — Mode "coller un brouillon"

`AskUserQuestion` avec texte libre :

> "Colle ici le contenu markdown complet du body de ton skill (sans le frontmatter `---`, juste le contenu après)."

Utilise tel quel.

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

## Étape 7 — Construire les fichiers du nouveau skill

Crée les dossiers :
```
~/code/skills/plugins/<slug>/
├── .claude-plugin/
└── skills/<slug>/
```

Crée `~/code/skills/plugins/<slug>/.claude-plugin/plugin.json` :
```json
{
  "name": "<slug>",
  "description": "<description complète frontmatter>",
  "version": "1.0.0",
  "author": {
    "name": "Samuel Audette",
    "url": "https://github.com/RunLittleTurtle"
  },
  "license": "MIT",
  "homepage": "https://github.com/RunLittleTurtle/skills"
}
```

Crée `~/code/skills/plugins/<slug>/skills/<slug>/SKILL.md` :
```markdown
---
name: <slug>
description: <description>
---

<body construit à l'étape 6>
```

## Étape 8 — Mettre à jour `marketplace.json`

Lis `~/code/skills/.claude-plugin/marketplace.json` (JSON valide).

Ajoute une nouvelle entrée à la fin du tableau `plugins` :
```json
{
  "name": "<slug>",
  "source": "./plugins/<slug>",
  "description": "<description courte, 1 phrase>"
}
```

**Format `source` important** : utilise toujours `"./plugins/<slug>"` (chemin relatif explicite). Le format court `"<slug>"` avec `metadata.pluginRoot` n'est pas accepté par le validator actuel.

Préserve le formatting (2 espaces d'indentation, virgules correctes entre les entrées). Réécris le fichier complet.

Validation : après écriture, lance `claude plugin validate ~/code/skills` et résous toute erreur avant de continuer.

## Étape 9 — Mettre à jour `README.md`

Lis `~/code/skills/README.md`. Trouve la section `## Skills disponibles` et son tableau.

Ajoute une nouvelle ligne au tableau :
```
| `<slug>` | <description courte 1 phrase> | <use case 1 phrase> |
```

Si le tableau n'existe plus ou a été modifié manuellement, dis-le à l'utilisateur et demande comment procéder.

## Étape 10 — Commit + push ?

`AskUserQuestion` :

- **Oui, commit + push maintenant** :
  ```bash
  cd ~/code/skills
  git add -A
  git commit -m "Add <slug> skill"
  git push
  ```
- **Non, je m'en charge plus tard** : affiche le chemin local et les commandes à exécuter à la main.

## Étape 11 — Récap final

Affiche :

```
✓ Skill <slug> créé.

Fichiers :
  ~/code/skills/plugins/<slug>/
  ├── .claude-plugin/plugin.json
  └── skills/<slug>/SKILL.md

[Si pushé]
GitHub : https://github.com/RunLittleTurtle/skills/tree/main/plugins/<slug>

Pour l'installer dans Claude Code :
  /plugin marketplace update skills
  /plugin install <slug>@skills

Slash command résultant : /<slug>:<slug>

[Si pas pushé]
À faire plus tard :
  cd ~/code/skills && git add -A && git commit -m "Add <slug> skill" && git push
```

## Notes importantes pour Claude pendant l'exécution

- **Ne crée jamais de symlink**. Le skill vit uniquement dans `~/code/skills/plugins/<slug>/`. L'installation côté utilisateur passe par `/plugin install`, pas par un symlink manuel.
- **Ne crée pas de nouveau repo GitHub**. Tout va dans le mono-repo `RunLittleTurtle/skills`. Pas de `gh repo create`.
- **Ne modifie jamais les autres skills existants**. Touche uniquement aux fichiers du nouveau skill + `marketplace.json` + `README.md`.
- **Si une étape échoue** (pull conflict, JSON invalide, slug en doublon), arrête le workflow et dis à l'utilisateur exactement ce qui s'est passé. Ne tente pas de réparer en silence.
- **Frontmatter SKILL.md** : exactement `name` + `description` (standard agentskills.io). Ne pas ajouter `version`, `tags` ou autres champs non standard.
