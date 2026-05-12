---
name: bmad-customize-skills
description: Désactiver ou réactiver sélectivement les skills BMad-Method dans un projet (preset Product-only ou sélection custom par catégorie). Déplace les dossiers de skills BMad hors du chemin scanné et patche bmad-help.csv pour neutraliser les required gates des phases non pertinentes. Idempotent, réversible (reset all), et ré-applicable après chaque `npx bmad-method install` (détecte le drift). À invoquer quand l'utilisateur veut adapter une install BMad à un projet non-coding (doc, process mapping), ou plus largement quand il veut désactiver des skills BMad précis pour réduire le bruit du flow guidé.
---

# bmad-customize-skills

Adapte une install BMad à un projet réel en désactivant proprement les skills non pertinents — sans casser BMad ni perdre la configuration après une réinstallation.

**Réponds en français. Sois concis. Confirme chaque action avant de l'exécuter.**

---

## Étape 1 — Détecter l'installation BMad

Cherche `_bmad/_config/bmad-help.csv` dans le répertoire courant. S'il n'existe pas, remonte jusqu'à 5 niveaux parents avec :

```bash
dir="$(pwd)"; for i in 1 2 3 4 5; do
  [ -f "$dir/_bmad/_config/bmad-help.csv" ] && echo "FOUND: $dir" && break
  dir="$(dirname "$dir")"
done
```

Si introuvable, **arrête** avec ce message :

> Aucune installation BMad détectée à partir d'ici. Ce skill modifie des projets contenant `_bmad/`. Lance `npx bmad-method install` dans le projet cible, puis ré-invoque ce skill depuis ce dossier.

Sinon, retiens ce chemin comme `<bmad_root>`. Tous les chemins relatifs ci-dessous y sont ancrés. Les 3 dossiers de déploiement des skills sont :
- `<bmad_root>/.claude/skills/` (Claude Code)
- `<bmad_root>/.agents/skills/` (Codex / Cursor / Gemini / OpenCode)
- `<bmad_root>/.agent/skills/` (Antigravity)

---

## Étape 2 — Charger l'état courant

Lis `<bmad_root>/_bmad/custom/bmad-customize-state.json` s'il existe (avec `Read`). Sinon, l'état initial est :

```json
{
  "version": 1,
  "last_applied": null,
  "preset_used": null,
  "disabled_skills": [],
  "csv_rows_modified": {},
  "bmad_install_fingerprint": null,
  "skill_version": "1.0.0"
}
```

Calcule le fingerprint actuel du CSV (SHA-256) :
```bash
shasum -a 256 "<bmad_root>/_bmad/_config/bmad-help.csv" | awk '{print $1}'
```

---

## Étape 3 — Détecter le drift post-install

Si `state.bmad_install_fingerprint` existe ET diffère du fingerprint actuel ET `state.disabled_skills` n'est pas vide :

- Pour chaque skill `S` dans `state.disabled_skills`, vérifie si `<bmad_root>/.claude/skills/<S>/` existe à nouveau.
- Si au moins un revenant est détecté, affiche :

> ⚠️ BMad a été réinstallé depuis le dernier `bmad-customize-skills`. N skills désactivés sont réapparus dans `.claude/skills/`. Je peux ré-appliquer ta configuration en un clic.

Et propose immédiatement le choix `[Re-apply current state]` dans le menu (Étape 5).

---

## Étape 4 — Afficher l'état courant

Affiche en table :

| Champ | Valeur |
|---|---|
| Preset actif | `state.preset_used` ou "aucun" |
| Skills désactivés | N (avec liste tronquée à 5) |
| Rows CSV modifiées | M |
| Dernière application | `state.last_applied` ou "jamais" |
| Drift détecté | oui/non |

---

## Étape 5 — Menu principal

Utilise `AskUserQuestion` avec ces options :

1. **Apply Product-only preset** — désactive automatiquement tout ce qui est dev / architecture / sprint (voir Étape 6a)
2. **Custom selection** — sélection multi-catégories avec descriptions par skill (Étape 6b)
3. **Modify existing** — ajuste la liste actuelle (cocher/décocher) (Étape 6c)
4. **Re-apply current state** — ré-applique le state existant (utile après `npx bmad-method install`)
5. **Reset all** — restaure les 52 skills par défaut + restaure le CSV (Étape 6d)
6. **View only** — pas d'action, affiche juste l'état détaillé puis termine
7. **Cancel** — quitte sans rien faire

---

## Étape 6a — Apply Product-only preset

**Liste exacte des skills désactivés par Product-only** (16 skills) :

Catégorie *Dev / Code execution* :
- `bmad-dev-story`
- `bmad-code-review`
- `bmad-quick-dev`
- `bmad-qa-generate-e2e-tests`
- `bmad-checkpoint-preview`

Catégorie *Architecture & Implementation* :
- `bmad-create-architecture`
- `bmad-create-epics-and-stories`
- `bmad-check-implementation-readiness`
- `bmad-sprint-planning`
- `bmad-sprint-status`
- `bmad-create-story`
- `bmad-retrospective`
- `bmad-agent-architect`
- `bmad-agent-dev`

Catégorie *Utilitaires code-oriented* :
- `bmad-document-project`
- `bmad-generate-project-context`

Affiche cette liste exacte avant de continuer. Va ensuite à l'Étape 7 (safe list check) puis Étape 8 (dry-run).

---

## Étape 6b — Custom selection

Pose une `AskUserQuestion` **par catégorie** (multi-select). Pour chaque option, inclus le nom du skill ET sa description courte. Voici le catalogue :

### Catégorie 1 — Dev / Code execution
- `bmad-dev-story` — Exécute l'implémentation d'une story (red-green-refactor)
- `bmad-code-review` — Review de code adversarial multi-couches
- `bmad-quick-dev` — Workflow intent→code unifié
- `bmad-qa-generate-e2e-tests` — Génère tests E2E pour features existantes
- `bmad-checkpoint-preview` — Walkthrough humain d'un commit/branche/PR

### Catégorie 2 — Architecture & Implementation
- `bmad-create-architecture` — Crée le doc d'architecture technique
- `bmad-create-epics-and-stories` — Découpe les requirements en epics et stories
- `bmad-check-implementation-readiness` — Vérifie alignement PRD/UX/Archi/Stories
- `bmad-sprint-planning` — Génère le plan d'implémentation suivi par les agents
- `bmad-sprint-status` — Résume statut sprint et risques
- `bmad-create-story` — Prépare la prochaine story du sprint
- `bmad-retrospective` — Review de fin d'epic
- `bmad-agent-architect` — Agent Winston (System Architect)
- `bmad-agent-dev` — Agent Amelia (Senior Software Engineer)

### Catégorie 3 — CIS (Creative Intelligence Suite)
- `bmad-cis-agent-storyteller` — Agent Sophia (Master Storyteller)
- `bmad-cis-agent-design-thinking-coach` — Agent Maya (Design Thinking)
- `bmad-cis-agent-brainstorming-coach` — Agent Carson (Brainstorming)
- `bmad-cis-agent-creative-problem-solver` — Agent Dr. Quinn (Problem Solver)
- `bmad-cis-agent-innovation-strategist` — Agent Victor (Innovation)
- `bmad-cis-agent-presentation-master` — Agent Caravaggio (Presentation)
- `bmad-cis-innovation-strategy` — Identifier opportunités de disruption
- `bmad-cis-problem-solving` — Méthodologies systématiques de problem-solving
- `bmad-cis-design-thinking` — Processus human-centered design
- `bmad-cis-storytelling` — Frameworks narratifs

### Catégorie 4 — Research & Discovery
- `bmad-market-research` — Analyse marché, compétition, clients
- `bmad-domain-research` — Deep dive domaine / industrie
- `bmad-technical-research` — Faisabilité technique
- `bmad-brainstorming` — Facilitation expert d'idéation
- `bmad-product-brief` — Création de brief produit
- `bmad-prfaq` — Working Backwards PRFAQ challenge

### Catégorie 5 — Reviews avancés
- `bmad-review-adversarial-general` — Cynical review générique
- `bmad-review-edge-case-hunter` — Hunt exhaustif d'edge cases
- `bmad-editorial-review-structure` — Review structurel (cuts, réorganisation)
- `bmad-editorial-review-prose` — Review de copy / prose

### Catégorie 6 — Utilitaires
- `bmad-shard-doc` — Découpe un gros markdown en sections
- `bmad-index-docs` — Génère un index.md
- `bmad-distillator` — Compression LLM lossless d'un doc
- `bmad-document-project` — Documente un codebase existant
- `bmad-generate-project-context` — Génère project-context.md depuis un codebase
- `bmad-correct-course` — Navigue les changements significatifs en sprint

Pour chaque catégorie, l'utilisateur coche ce qu'il veut **désactiver**. Concatène toutes les sélections. Va ensuite à l'Étape 7.

---

## Étape 6c — Modify existing

Affiche la liste actuelle de `state.disabled_skills`. Demande via `AskUserQuestion` :

1. **Ajouter des skills à désactiver** → branche vers Étape 6b (custom selection sur ce qui n'est pas déjà désactivé)
2. **Réactiver certains skills** → multi-select sur la liste actuelle
3. **Les deux** → enchaîner les deux
4. **Annuler**

Compute la liste finale souhaitée → va à l'Étape 7.

---

## Étape 6d — Reset all

Confirme avec `AskUserQuestion` :

> "Restaurer les 52 skills BMad par défaut ? Ça va déplacer tous les dossiers de `_bmad-disabled-skills/` retour dans `skills/` et restaurer le CSV. Action réversible (relance ce skill ensuite)."

Si oui :
- Pour chaque emplacement `.claude/`, `.agents/`, `.agent/` : `mv <root>/_bmad-disabled-skills/* <root>/skills/` (ignorer si dossier vide)
- Pour chaque entrée de `state.csv_rows_modified` : restaurer `required` à la valeur d'origine (clé = nom skill, valeur format `"true→false"` → restaurer à `true`)
- Supprimer (ou retirer) la colonne `_pruned` du CSV
- Écris un state vide + une ligne audit "Reset all" dans le log

Saute aux Étapes 11-12.

---

## Étape 7 — Safe list check (minimale)

**Skills protégés** (jamais désactivables, BMad casse sans eux) :

| Skill | Pourquoi |
|---|---|
| `bmad-help` | Sans lui, plus de navigation BMad ni de catalogue lisible |
| `bmad-customize` | Sans lui, plus aucun mécanisme d'override BMad |

Si l'utilisateur a sélectionné un de ces deux dans Étape 6b/6c, **refuse** avec :

> ❌ `<skill>` est protégé : <raison>. Sélection ignorée pour ce skill.

Et retire-le de la liste avant Étape 8.

Tous les autres skills (agents produit, create-prd, UX, etc.) sont désactivables au choix de l'utilisateur.

---

## Étape 8 — Dry-run (preview)

Calcule le diff complet :

- `to_disable` = skills à passer de `skills/` vers `_bmad-disabled-skills/`
- `to_enable` = skills à passer de `_bmad-disabled-skills/` vers `skills/`
- `csv_to_toggle` = rows du CSV à modifier (skill → `true→false` ou `false→true`)

Affiche en table :

```
Action       | Skill                          | Effet
-------------|--------------------------------|---------------------------
DISABLE      | bmad-dev-story                 | move + CSV required→false
DISABLE      | bmad-agent-architect           | move (pas dans le CSV)
ENABLE       | bmad-create-story              | move + CSV required→true (restored)
...
Total: N actions
```

Demande via `AskUserQuestion` :
- **Apply** → continue Étape 9
- **Show full details** → affiche détails pour chaque ligne (chemins exacts, valeurs CSV avant/après) puis re-pose la question
- **Cancel** → abort sans rien faire

---

## Étape 9 — Execute filesystem

Pour chaque skill `S` dans `to_disable`, pour chaque emplacement `R` dans `[.claude, .agents, .agent]` :

```bash
mkdir -p "<bmad_root>/R/_bmad-disabled-skills"
if [ -d "<bmad_root>/R/skills/S" ]; then
  mv "<bmad_root>/R/skills/S" "<bmad_root>/R/_bmad-disabled-skills/S"
fi
```

Pour chaque skill `S` dans `to_enable`, inverse :

```bash
if [ -d "<bmad_root>/R/_bmad-disabled-skills/S" ]; then
  mv "<bmad_root>/R/_bmad-disabled-skills/S" "<bmad_root>/R/skills/S"
fi
```

Si une source n'existe pas, skip silencieusement (log dans l'audit). N'échoue pas le batch si un emplacement est désynchronisé.

---

## Étape 10 — Patcher `bmad-help.csv`

Lis `<bmad_root>/_bmad/_config/bmad-help.csv`. Le header est :

```
module,skill,display-name,menu-code,description,action,args,phase,after,before,required,output-location,outputs
```

Pour chaque skill `S` dans la liste finale désactivée, si une row a `skill == S` ET `required == true` :
- Edit (via tool `Edit`) la valeur `true` → `false` sur cette row spécifique
- Track dans `state.csv_rows_modified[S] = "true→false"`

Pour les réactivations (skills réactivés), inverse via `state.csv_rows_modified[S]` :
- Si valeur = `"true→false"`, restaurer à `true`
- Supprimer l'entrée du tracking

**Colonne `_pruned`** : ajoute une 14e colonne en fin de ligne (`_pruned`) valeur `true` pour les rows désactivées, `false` ou vide pour les autres. Modifie aussi le header. **Skip cette étape si la colonne existe déjà** — patche juste les valeurs.

Préserve l'encodage UTF-8 et le séparateur `,`.

---

## Étape 11 — Persister état + audit

### State JSON

Écris `<bmad_root>/_bmad/custom/bmad-customize-state.json` :

```json
{
  "version": 1,
  "last_applied": "<ISO-8601 now>",
  "preset_used": "product-only|custom|null",
  "disabled_skills": ["bmad-dev-story", "bmad-agent-architect", ...],
  "csv_rows_modified": {
    "bmad-create-architecture": "true→false",
    ...
  },
  "bmad_install_fingerprint": "<nouveau SHA-256 du CSV après patch>",
  "skill_version": "1.0.0"
}
```

`mkdir -p <bmad_root>/_bmad/custom` si nécessaire.

### Audit log Markdown

Écris (ou append à) `<bmad_root>/_bmad/custom/BMAD_CUSTOMIZE_SKILLS_LOG.md` :

```markdown
# Journal `bmad-customize-skills`

Ce fichier trace les actions du skill `bmad-customize-skills` dans ce projet.
Source de vérité technique : `bmad-customize-state.json` (à côté de ce fichier).

## Configuration actuelle

- **Preset** : <product-only|custom|aucun>
- **Dernière application** : <ISO-8601>
- **Skills désactivés** : N
- **Required gates CSV neutralisées** : M

### Skills désactivés (par catégorie)

**Dev / Code execution**
- `bmad-dev-story`
- ...

**Architecture & Implementation**
- ...

## Comment réactiver

Re-lance le skill et choisis `Reset all` pour tout restaurer, ou `Modify existing` pour ajuster.

## Re-run après `npx bmad-method install`

L'installeur BMad regénère les skills à chaque exécution. Ce skill détecte le drift automatiquement : invoque-le, il proposera `Re-apply current state` sans questions.

## Historique

- **<ISO-8601>** — <preset> appliqué : N skills désactivés
- **<ISO-8601>** — Reset all
- ...
```

Append une ligne dans la section "Historique" pour la dernière action.

---

## Étape 12 — Validation finale

Vérifications :

1. Compter les rows `required=true` restantes dans le CSV (`grep -c ',true,' <csv>` approximatif — utilise plutôt une parse propre).
2. Vérifier que `bmad-help/` et `bmad-customize/` existent toujours dans `.claude/skills/` (safe list intacte).
3. Vérifier que tous les `state.disabled_skills` sont absents de `.claude/skills/` et présents dans `.claude/_bmad-disabled-skills/`.

Affiche le récap final :

```
✓ bmad-customize-skills terminé

  Preset appliqué    : <product-only|custom|reset>
  Skills désactivés  : N
  Required gates CSV : M (toggle vers false)
  Audit              : _bmad/custom/BMAD_CUSTOMIZE_SKILLS_LOG.md
  State              : _bmad/custom/bmad-customize-state.json

  Prochaine étape    : invoque /bmad-help pour vérifier le flow résultant.
```

---

## Notes importantes pour Claude pendant l'exécution

- **Tout est inline** — pas de bundled script externe.
- **3 emplacements traités indépendamment** (`.claude/`, `.agents/`, `.agent/`). Ne fais pas échouer le batch si un emplacement est manquant.
- **Édition CSV** : utilise `Read` puis `Edit` ligne-par-ligne avec contexte unique (jamais `replace_all`). Le CSV peut avoir des virgules dans les descriptions — fais attention aux faux positifs si tu utilises grep, préfère parse champ par champ.
- **Safe list non-négociable** : refuse même si l'utilisateur insiste. C'est ce qui empêche l'utilisateur de se tirer une balle dans le pied.
- **Toujours dry-run avant execute** — pas de raccourci, même pour Reset all (confirmation explicite suffit pour Reset).
- **`_bmad/custom/` survit aux réinstalls BMad** — c'est pourquoi on y stocke `state.json` et l'audit log. Ne stocke rien d'autre ailleurs.
- **Si Claude Code charge encore un skill malgré son déplacement** (cas pathologique non testé) : c'est probablement parce que Claude Code scanne récursivement. Le user doit relancer la session ; documente ça dans la sortie finale.
