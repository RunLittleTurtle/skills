# use-case-value (v1.1)

> Priorise les use cases d'AI/automatisation par **impact business chiffré documenté**, à partir de matériel d'atelier de découverte (transcripts, post-its, inventaires de processus, notes de réunion). Six sources d'Impact $ explicites (temps perdu, opportunités manquées, coût des erreurs, saturation main-d'œuvre, dépendances externes, frictions inter-départementales) + colonne `Dépend de` pour modéliser les dépendances inter-use-cases + Step 2bis Root Cause Analysis (5 Whys / Ishikawa) avant chiffrage. Règle d'or stricte : seuls les chiffres durs cités ou validés par le sponsor entrent dans les cellules Impact $ ; les estimations LLM vont exclusivement en Notes comme questions à poser au sponsor. Synthèse one-pager en prose narrative pour directeurs de compte, avec section optionnelle Dépendances et root causes. **Aucune dimension d'effort technique** : ce skill se concentre sur la VALEUR uniquement, l'effort sera traité par un futur skill séparé.

## Nouveautés v1.1 (vs v1.0)

- **Step 2bis Root Cause Analysis** dans le workflow : avant chiffrage, le LLM applique un mini 5 Whys ou un Ishikawa léger sur chaque candidat use case pour distinguer **root cause actionnable** (à enregistrer comme ligne du CSV) vs **symptôme** (à consolider comme Pain Point sur la ligne de la root cause). Évite de triple-compter l'impact quand plusieurs symptômes ont une seule automatisation possible.
- **Col 26 `Dépend de`** : texte libre qui liste les use cases prérequis identifiés en Step 2bis. Permet une lecture mécanique des chaînes (ex : 3 use cases du Top dépendent du Data Lake en amont). Cette colonne n'entre pas dans le calcul du Score Priorité Impact, elle sert de signal qualitatif pour la synthèse et la planification de l'ordre de lancement.
- **Section optionnelle "Dépendances et root causes"** dans la synthèse : 2-3 paragraphes en prose qui identifient les chaînes critiques et les fondations à débloquer en premier. Incluse uniquement si pertinent (col 26 a des valeurs ou Step 2bis a consolidé des symptômes), omise sinon pour préserver le format one-pager pour les cas simples.
- Sources méthodologiques étendues : ajout de **5 Whys (Taiichi Ohno, Toyota Production System)** et **Ishikawa Diagram (Kaoru Ishikawa)** pour Step 2bis. **SAFe Enabler** pour la notion de dépendance infrastructure.
- CSV passe de 25 à 26 colonnes ; le workflow ajoute une Step 2bis sans casser les autres étapes.

## Use case

Tu as fini un atelier de découverte (transcripts, post-its, inventaires) et tu dois décider quel use case mérite d'être chiffré et scopé en premier sur la base de l'impact business **réellement documenté** chez ton client. Le skill transforme le matériel brut en deux livrables prêts à partager : un CSV de travail 25 colonnes (avec FORMULES et SOURCE pour traçabilité) et une synthèse exécutive en prose narrative pour les directeurs de compte.

## Pourquoi ce skill et pas `use-case-prioritization` v2.2 ?

`use-case-prioritization` v2.2 calcule un ROI complet (impact + effort + payback + verdicts hybrides) mais a deux limites observées en usage réel terrain :

1. **Le modèle de Bénéfice Net est trop étroit** : il repose principalement sur Heures × Coût horaire, ce qui sous-estime massivement les use cases dont la valeur vient des opportunités commerciales, des erreurs absorbées en marge, ou de la saturation de ressources critiques. Un use case "Soumissions clients" peut classer rang 9 mécanique alors qu'il vaut 200 000 $ dans la réalité.

2. **Le CSV à 38 colonnes et la synthèse à 5 sections deviennent illisibles** pour un directeur de compte non technique. Trop de chiffres par phrase, jargon de bundling LLM, perte du fil narratif.

`use-case-value` adopte une approche différente : **séparation de la lecture impact et de la lecture effort**. Ce skill se concentre uniquement sur l'impact, avec 6 sources de valeur explicites et une règle d'or stricte sur l'absence d'estimations LLM dans les cellules. L'effort sera traité par un futur skill séparé qui pourra se brancher sur le CSV de sortie pour fermer la lecture ROI.

Le plugin v2.2 `use-case-prioritization` reste disponible en parallèle ; les deux skills coexistent dans la marketplace.

## Règle d'or — chiffres durs uniquement dans les cellules

Pour éviter que les estimations LLM viennent gonfler les vraies données :

- **Cellules Impact $ (cols 13-18)** : uniquement des valeurs citées par un sponsor / participant atelier, ou dérivées par formule simple sur ces chiffres durs (ex : "20-30 heures par semaine" × 50 sem × 80 $/h fully-loaded). Sinon : 0 ou vide.
- **Colonne Notes (col 25)** : les estimations indicatives, sous forme structurée :
  ```
  À chiffrer en atelier :
  - Combien de contrats perdus par an à cause du délai 3 semaines ? À titre
    indicatif, si 5 contrats par an × 50 000 $ × 40 % probabilité capture,
    +100 000 $ en Impact Opportunités.
  ```
  Ce sont des questions à poser au sponsor, pas des données à utiliser pour le verdict.

Conséquence : un use case "vraiment important mais mal chiffré" classera initialement avec un Verdict "À chiffrer prioritairement". C'est honnête et ça aiguille le travail vers la bonne prochaine étape (atelier de validation avec le sponsor).

## Installation

### Pour Claude Code

```
/plugin marketplace add RunLittleTurtle/skills
/plugin install use-case-value@skills
```

Mise à jour : `/plugin marketplace update skills`. Désinstallation : `/plugin uninstall use-case-value@skills`.

### Pour les autres outils (Claude Desktop, OpenCode, Cursor, GitHub Copilot, Gemini CLI, Goose, etc.)

Standard ouvert [agentskills.io](https://agentskills.io). Clone le repo et copie le skill dans le dossier skills de ton outil :

```bash
git clone https://github.com/RunLittleTurtle/skills.git
cp -R skills/plugins/use-case-value/skills/use-case-value <DOSSIER_SKILLS_DE_TON_OUTIL>/
```

| Outil | Dossier cible |
|---|---|
| Claude Code | `~/.claude/skills/` |
| Claude Desktop (macOS) | `~/Library/Application Support/Claude/skills/` |
| OpenCode | `~/.opencode/skills/` |
| Cursor | voir [docs Cursor skills](https://cursor.com/docs/context/skills) |
| GitHub Copilot | voir [docs Copilot agent skills](https://docs.github.com/en/copilot/concepts/agents/about-agent-skills) |

## Invocation

Slash command Claude Code : `/use-case-value:use-case-value` (ou `/use-case-value` selon ta config).

Triggers détectés : `prioriser par impact`, `valeur use case`, `quel use case en premier`, `chiffrer la valeur`, `note de cadrage`, `atelier découverte`, `post-its`, `transcript atelier`, `inventaire processus`, `impact business`, `pertes évitées`, `opportunités manquées`.

## Contenu

- `skills/use-case-value/SKILL.md` — workflow Step 0-7.
- `skills/use-case-value/references/framework.md` — explication des 25 colonnes, rubriques 1-3, sources méthodologiques (BABOK Business Case, WSJF, DAMA-DMBOK Data Availability).
- `skills/use-case-value/references/calculation_rules.md` — formules, défauts (coût horaire 80 $/h), grille de verdict.
- `skills/use-case-value/references/csv_template.csv` — gabarit 25 colonnes avec FORMULES + SOURCE + exemple fictif Soumissions Acme.
- `skills/use-case-value/references/synthese_template.md` — gabarit synthèse one-pager prose narrative.
- `skills/use-case-value/references/exemples_chiffrage_impact.md` — aide-mémoire pour distinguer chiffre dur vs hypothèse, exemples typiques.

## Licence

MIT — voir [LICENSE](../../LICENSE) à la racine du repo.
