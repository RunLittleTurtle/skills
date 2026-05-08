---
name: coordination
description: Coordonner plusieurs instances Claude qui travaillent en parallèle sur le même repo (typiquement plusieurs fenêtres iTerm). Crée ou rejoint un fichier de log par focus (sprint, task, prd, etc.) avec un système de locks logiques en markdown. À invoquer en début de session quand l'utilisateur veut faire travailler plusieurs Claude en parallèle sans collision sur les mêmes fichiers de doc.
---

# Coordination multi-instance Claude

Aide l'utilisateur à coordonner plusieurs instances Claude qui travaillent en parallèle dans différentes fenêtres iTerm sur le même repo. Le système utilise des fichiers markdown par focus (`COORDINATION.<FOCUS>.md`) avec des locks advisory et un protocole partagé.

**Réponds en français. Sois concis. Confirme les actions en une phrase.**

## Étape 1 — Vérifications préalables

Exécute `git rev-parse --show-toplevel`. Si ça échoue, dis "Pas dans un repo git, le skill ne peut pas continuer" et arrête. Sinon, mémorise le résultat comme `REPO_ROOT` pour la suite.

## Étape 2 — Détection de l'état actuel

En parallèle, vérifie l'état du repo :
- Existence de `<REPO_ROOT>/COORDINATION.README.md`
- Liste des `<REPO_ROOT>/COORDINATION.*.md` SAUF `COORDINATION.README.md` (utilise `ls` ou `find`)
- Présence du pattern `COORDINATION.*.md` dans `<REPO_ROOT>/.gitignore` (lis le fichier si existe)
- Présence d'une section `## Multi-instance coordination` dans `<REPO_ROOT>/CLAUDE.md`

Note : `COORDINATION.example.md` matche le wildcard, ignore-le aussi dans le scan des focus files (c'est un artefact de la phase de design).

## Étape 3 — Phase setup (si 1ère fois)

**Condition** : si README absent OU le pattern n'est pas dans .gitignore OU la note n'est pas dans CLAUDE.md.

Présente à l'utilisateur **exactement** ce qui sera fait, puis utilise `AskUserQuestion` :

> *"Setup initial requis. Le skill va :*
> *1. Créer `COORDINATION.README.md` (protocole, committé)*
> *2. Ajouter `COORDINATION.*.md` au `.gitignore` (avec exclusion du README)*
> *3. Ajouter une note `## Multi-instance coordination` dans `CLAUDE.md`*
>
> *Procéder au setup ?"*

Options : "Oui, procéder" / "Non, annuler".

**Si l'utilisateur annule** : dis "Setup annulé, le skill ne peut pas fonctionner sans" et arrête.

**Si oui**, fais les actions manquantes (et seulement les manquantes, pour idempotence) :

### 3a. Créer `COORDINATION.README.md` si absent

Écris ce contenu (utilise Write) :

```markdown
# COORDINATION.README.md

> Protocole et légende du système de coordination multi-instance Claude pour ce repo.
> Pour activer la coordination dans une session, utilise la commande `/coordination` au début de session.

## À quoi sert ce système

Coordonner plusieurs instances Claude (et toi, l'humain) qui travaillent en parallèle sur ce repo, pour éviter qu'elles écrasent mutuellement leurs édits sur les mêmes fichiers.

**Lecture, recherche, planification : LIBRE**, pas besoin de lock.
**Edit/Write sur un fichier partagé : OBLIGATOIRE** d'écrire une entrée `[LOCKED]` dans le log avant.

## Architecture multi-focus

Un fichier de log par **focus** (sujet/projet) :
- `COORDINATION.SPRINT.md` — instances qui travaillent sur les sprints
- `COORDINATION.TASK.md` — instances qui travaillent sur les tâches/user stories
- `COORDINATION.PRD.md` — instances qui travaillent sur les PRD
- ... (autres focus créés à la volée selon besoin)

Une instance ne lit/n'écrit QUE dans le fichier correspondant à son focus. Les instances sur des focus différents ne se voient pas — par convention, elles ne touchent pas aux mêmes fichiers de doc.

## Statuts d'une entrée

| Statut | Signification | Action requise par les autres |
|---|---|---|
| `[INTENT]` | Planifié, pas commencé | Aucune (autres peuvent continuer) |
| `[LOCKED]` | En train d'écrire MAINTENANT | **Personne d'autre n'écrit sur ce fichier/section** |
| `[WAITING]` | Veut écrire mais bloqué par un `[LOCKED]` actif | Attend que le lock se libère |
| `[DONE]` | Terminé | Aucune (servira de contexte aux suivants) |
| `[CANCELLED]` | Abandonné | Aucune |

**Quand le statut change**, on **modifie la ligne sur place**, on ne la déplace PAS. La ligne reste à sa position chronologique.

## Format d'une entrée (Claude structuré)

```
- [STATUS] <slug> by <inst-id> (<timing>) — <fichier> §<section> — "<intent ou résultat>"
```

- **`<slug>`** : court, kebab-case (ex: `bilan-9`, `meetings-section`)
- **`<inst-id>`** : `inst-A`, `inst-B`, … ou `human` pour l'humain
- **`<timing>`** :
  - `[LOCKED]` : `claimed HH:MM, TTL 30min`
  - `[DONE]` : `HH:MM → HH:MM` (début → fin)
  - `[INTENT]` / `[WAITING]` : `added HH:MM`
- **`<section>`** : `§Bilan`, `§3.2`, `L12` (ligne 12), ou `tout` si fichier entier
- **`"<…>"`** : intention courte si en cours, résultat court si fini

## Format simplifié (humain)

L'humain peut écrire plus rudimentaire. Tant que la ligne commence par `[STATUS]`, c'est OK :

```
- [DONE] human: ajouté date rétro sprint 9 ligne 12 (14:42)
```

## Règles d'usage

1. **Lecture/recherche/planification** : LIBRE, pas besoin de claim.
2. **Edit/Write** : ajoute une ligne `[LOCKED]` AVANT, change-la en `[DONE]` APRÈS.
3. **Avant de claim** : lis le log, vérifie qu'aucun `[LOCKED]` ou `[WAITING]` non périmé ne couvre ta zone. Ignore les locks dont `claimed + TTL < maintenant` (= périmés).
4. **Sur collision** (un autre lock couvre ta zone) : ajoute une ligne `[WAITING]` et **demande à l'humain** quoi faire (attendre, switcher, abandonner).
5. **Quand un `[LOCKED]` que tu attendais devient `[DONE]`** : RELIS le fichier modifié, valide que ton plan tient toujours, et **demande à l'humain "OK go ?"** avant de claim.
6. **Toi (humain)** : tu peux écrire ici aussi, owner = `human`. Si tu vois un lock zombie (TTL dépassé), tu peux le passer en `[CANCELLED]` à la main.

## Quoi grep pour quoi

| Tu veux savoir… | Cherche… |
|---|---|
| Qui écrit MAINTENANT | `[LOCKED]` |
| Qui veut écrire mais bloqué | `[WAITING]` |
| Ce qui est planifié mais pas commencé | `[INTENT]` |
| Ce qui a été terminé récemment | `[DONE]` |
| Toute l'activité sur un fichier précis | `<nom-du-fichier>` |
| Toute l'activité d'une instance | `inst-A` (ou autre owner) |
```

### 3b. Mettre à jour `.gitignore` si pattern absent

Lis le `.gitignore` (si existe). Vérifie si le pattern `COORDINATION.*.md` est déjà présent. Si oui, ne touche pas. Si non, append à la fin :

```
# Coordination logs (ephemeral, per focus)
COORDINATION.*.md
!COORDINATION.README.md
```

Si `.gitignore` n'existe pas, crée-le avec ce bloc.

### 3c. Ajouter la note dans `CLAUDE.md` si absente

Lis `CLAUDE.md`. Cherche la section `## Multi-instance coordination`. Si elle existe, ne touche pas. Si non, append à la fin du fichier :

```markdown

## Multi-instance coordination

Pour coordonner plusieurs instances Claude qui travaillent en parallèle sur ce repo, utilise la commande `/coordination` au début de chaque session. Voir `COORDINATION.README.md` pour le protocole complet.
```

### Confirmation du setup

Dis en une phrase ce qui a été fait : *"Setup terminé : README créé, .gitignore mis à jour, note ajoutée à CLAUDE.md."* (ou seulement les éléments effectivement modifiés).

## Étape 4 — Scanner les focus existants

Liste tous les fichiers `<REPO_ROOT>/COORDINATION.*.md` SAUF :
- `COORDINATION.README.md`
- `COORDINATION.example.md` (artefact du design, ignorer)
- `COORDINATION.*.bak.*.md` (archives, ignorer)

Pour chaque fichier focus trouvé :
1. Extrais le nom du focus depuis le filename (ex: `COORDINATION.SPRINT.md` → `SPRINT`)
2. Lis la dernière ligne d'activité commençant par `- [` (la dernière entrée du log)
3. Extrais le timestamp si présent dans cette ligne
4. Compte les lignes `[LOCKED]` et `[WAITING]` actives (ignore les `[LOCKED]` dont `claimed + TTL < maintenant`)

Garde ces données pour les questions suivantes.

## Étape 5 — Question principale : que faire ?

Présente les options via `AskUserQuestion` selon ce qui existe :

### Cas A — Aucun focus file n'existe

Question : *"Aucun coordination log n'existe encore. Que veux-tu faire ?"*

Options :
- **"Créer un nouveau focus"** (Recommended) — Démarrer un log de coordination pour un nouveau sujet
- **"Ne rien faire"** — Sortir du skill, travailler en mode normal sans coordination

### Cas B — Au moins un focus file existe

Question : *"Coordination active dans ce repo. Que veux-tu faire ?"*

Options :
- **"Se connecter à un focus existant"** (Recommended) — Rejoindre une session de coordination en cours
- **"Créer un nouveau focus"** — Démarrer un nouveau log pour un sujet différent
- **"Clear un focus existant"** — Archiver et recréer un log (action destructive)
- **"Ne rien faire"** — Sortir sans coordination

## Étape 6 — Action selon le choix

### "Ne rien faire"

Dis "Pas de coordination pour cette session, tu travailles en mode normal." et termine le skill. Ne charge PAS le protocole en contexte.

### "Créer un nouveau focus"

1. **D'abord, analyse le contexte de la conversation actuelle** :
   - Quels fichiers ont été lus, édités ou mentionnés récemment ?
   - Quels sujets l'utilisateur a évoqué (sprints, PRD, user stories, refonte, recherche, etc.) ?
   - Y a-t-il un thème dominant clair ?
   - Les premiers messages de l'utilisateur dans cette session donnent souvent l'indice le plus fort.

2. **Si un focus se dégage clairement du contexte** :
   - **IMPORTANT : le focus doit être UN SEUL MOT en UPPERCASE** qui décrit le SUJET général (pas une tâche précise). Les détails de tâche iront dans les `<slug>` des entrées du log, pas dans le nom du fichier.
   - Exemples corrects : `SPRINT`, `TASK`, `PRD`, `RESEARCH`, `REFONTE`, `STORIES`
   - Exemples incorrects : `SPRINT-9`, `USER-STORIES-SECTION-3`, `REFONTE-PRD-RAG` (ce sont des slugs d'entrée, pas des focus)
   - Propose le mot déduit. Exemple : *"Vu qu'on travaille sur les user stories, je suggère focus = `STORIES` ou `TASK`."*
   - Utilise `AskUserQuestion` avec 2 options :
     - "Garder `<MOT-DÉDUIT>`" (Recommended)
     - "Renommer" (le user clique "Other" pour taper son propre nom)

3. **Si aucun contexte clair** (session fraîche, aucun fichier touché, conversation vague) :
   - **Ne propose PAS de noms hardcodés comme `Sprint`/`Task`/`PRD`.**
   - Demande directement en texte libre dans le chat (PAS de `AskUserQuestion`) : *"Sur quoi vas-tu travailler dans cette session ? Donne UN seul mot pour le sujet général (ex: `sprint`, `task`, `prd`, `research`, `refonte`). Pas une tâche précise — les détails iront dans les entrées du log."*
   - Attends la réponse de l'utilisateur dans le chat normal.

4. **Sanitize le nom reçu** :
   - Convertir en UPPERCASE
   - Garder uniquement alphanumérique
   - **Si plusieurs mots** : préviens l'utilisateur que la convention idéale est UN seul mot, et propose une version courte (ex: `"user research"` → suggère `RESEARCH` au lieu de `USER-RESEARCH`). Si l'utilisateur insiste sur plusieurs mots, OK, accepte avec dash : `USER-RESEARCH`. Mais c'est l'exception.
   - Exemples : `"sprint"` → `SPRINT` ✅, `"refonte"` → `REFONTE` ✅, `"user research"` → propose `RESEARCH`, fallback `USER-RESEARCH`

5. Construis le chemin : `<REPO_ROOT>/COORDINATION.<FOCUS>.md`. Vérifie qu'il n'existe pas déjà. S'il existe, dis à l'utilisateur et propose : (a) se connecter au focus existant, (b) choisir un autre nom.

6. Suggère `inst-A` comme identité (premier sur ce focus). Demande confirmation via `AskUserQuestion` : *"Identité pour cette session : `inst-A`. Garder ou renommer ?"*. Options : "Garder inst-A" / "Renommer". Si renommer, l'utilisateur tape via "Other".

7. Récupère l'heure courante en HH:MM (par exemple via `date +%H:%M`).

8. **Analyse le contexte de la conversation actuelle pour générer un RECAP structuré** :

   Le recap est un **briefing rapide** pour qu'une instance qui rejoindra plus tard puisse ramp up en lisant 4 lignes — pas un rapport détaillé. Si une instance veut le détail, elle ouvrira les fichiers eux-mêmes.

   **Format strict** :

   ```
   - [DONE] recap by <inst-id> (<HH:MM>) :
     - Sujet : <UNE phrase, max ~15 mots>
     - Files/sources : <liste de paths séparés par ` ; `>
     - Done : <items courts séparés par ` ; `>
     - Next : <prochaine action en 1 ligne>
   ```

   **Règles strictes de brièveté** :
   - **Chaque section = UNE ligne max.** Pas de sous-bullets, pas de paragraphes multi-lignes.
   - Plusieurs items dans une section : séparés par `;` (semicolon).
   - **Pas de stats détaillées, pas de chiffres précis** dans la recap. Juste le résultat-clé d'une étape ("RL-09 validé"), pas les pourcentages.
   - **Paths concrets et exacts** (relatifs au repo ou avec `~/` si externe). Pas de descriptions vagues.
   - Si une section n'a rien à mettre, **omettre la ligne** (ne pas mettre "aucun").
   - Total recap = max **5-6 lignes**. Si tu écris plus, c'est trop verbeux.

   **Exemple BON (concis)** :
   ```
   - [DONE] recap by inst-A (11:41) :
     - Sujet : validation règles RL-XX (doc v7.8) contre BD SQLite (81 RT)
     - Files/sources : Roadmap_projets/projet_4/Doc_fonctionnelle/regle_et_logique_v7.8.md ; ~/authentik-roadtrips/roadtrips.db ; Roadmap_projets/projet_4/Analyse_modeles_authentik/
     - Done : Glossaire Catalogue corrigé (81 RT) ; RL-02 réécrit ; RL-08 validé partiel ; RL-09 validé
     - Next : RL-10 TRANSIT (puis RL-11, RL-12, RL-14)
   ```

   **Exemple MAUVAIS (trop verbeux)** :
   ```
   - Done :
     - Glossaire `Catalogue` corrigé : 81 RT (49 CA + 32 US), pas 73. Stats HEX dans les règles restent basées sur snapshot 73 RT du 2026-04-28.
     - RL-02 réécrit avec décomposition 3 catégories : 48 % repartent dès le lendemain, 48 % restent exactement 2 nuits, 4 % restent 3+ nuits.
     [...]
   ```
   Trop de chiffres, sous-bullets, détails. Ce niveau de détail va dans les fichiers de travail, pas dans la recap.

   **Si la session est vraiment vide** (Claude vient de démarrer, aucun fichier touché, aucune décision) : skip cette étape et passe directement à 9 sans recap.

9. Crée le fichier avec ce template (remplace `<FOCUS>`, `<inst-id>`, `<HH:MM>`, et le bloc `<RECAP>`) :

   ```markdown
   # COORDINATION.<FOCUS>.md

   > Protocole & légende : voir `COORDINATION.README.md`
   > Ce fichier coordonne les instances avec **focus = <focus en lowercase>** uniquement.

   ---

   ## Contributeurs (focus: <focus en lowercase>)

   | Instance | Joined | Notes |
   |---|---|---|
   | <inst-id> | <HH:MM> | (ajouter notes si besoin) |
   | human | (toujours) | edits manuels ad hoc |

   **Lettres utilisées sur ce focus** : <lettre de inst-id> → prochaine instance qui rejoint = `inst-<lettre+1>`

   ---

   ## Activité (ancien → récent, ajout en bas)

   - [DONE] recap by <inst-id> (<HH:MM>) :
     - Sujet : <sujet>
     - Files/sources : <files>
     - Done : <done>
     - Next : <next>
   - [DONE] <inst-id> joined <FOCUS> coordination (<HH:MM>)
   ```

   **Important** : si pas de recap (session vide), n'inclus PAS le bloc recap. Mets seulement la ligne joined.

10. Confirme : *"Focus `<FOCUS>` créé avec recap du contexte. Tu es `<inst-id>` sur ce log."* (ou *"sans recap"* si session vide).

11. Passe à l'étape 7 (chargement du protocole).

### "Se connecter à un focus existant"

1. Si plusieurs focus existent, demande lequel via `AskUserQuestion` (2e question). Options format : `<FOCUS> (last activity HH:MM, N locks actifs)`. Si aucune activité depuis création : `<FOCUS> (no activity yet)`. Si N=0 : ne mentionne pas les locks. **Une option par focus existant.** L'option "Other" auto-fournie permet à l'utilisateur de taper un nom de focus s'il pointait vers un autre (rare).

   Si un seul focus existe, skip la question et utilise-le directement.

2. Lis `<REPO_ROOT>/COORDINATION.<FOCUS>.md` en entier.

3. Parse le tableau "Contributeurs" pour extraire les `inst-X` déjà utilisées. Calcule la prochaine lettre dispo (max + 1). Exemple : si A, B, C utilisées → suggérer `inst-D`.

4. Parse les `[LOCKED]` et `[WAITING]` actuels. Pour chaque `[LOCKED]`, vérifie si `claimed + TTL < maintenant` → marque comme périmé.

5. Affiche un résumé bref :
   - "Focus `<FOCUS>` : <N> instances enregistrées"
   - "Locks actifs : <liste courte> (ou aucun)"
   - "Locks périmés détectés : <liste courte>" (s'il y en a — l'utilisateur peut les nettoyer manuellement plus tard)
   - "Dernière activité : <description ligne>"

6. Suggère `inst-<lettre>` comme identité. Demande confirmation via `AskUserQuestion`. Options : "Garder inst-<lettre>" / "Renommer".

7. Récupère HH:MM courant.

8. Modifie le fichier (Edit) :
   - Append une ligne au tableau Contributeurs : `| <inst-id> | <HH:MM> | (focus actuel) |`
   - Mets à jour la ligne "Lettres utilisées" pour refléter la nouvelle lettre
   - Append une entrée au log : `- [DONE] <inst-id> joined <FOCUS> coordination (<HH:MM>)`

9. Confirme : *"Connecté à `<FOCUS>` comme `<inst-id>`. <résumé bref des locks actifs>."*

10. Passe à l'étape 7.

### "Clear un focus existant"

1. Si plusieurs focus, demande lequel via `AskUserQuestion`. Options format : `<FOCUS> (last activity HH:MM)`.

2. Demande confirmation explicite via `AskUserQuestion` : *"⚠️ Action destructive. Le fichier `COORDINATION.<FOCUS>.md` sera archivé en `.bak.<timestamp>.md` et recréé vide. Confirmer ?"*. Options : "Oui, clear" / "Non, annuler".

3. Si annulé, sors du skill.

4. Si confirmé :
   - Génère le timestamp `YYYYMMDD-HHMMSS` (via `date +%Y%m%d-%H%M%S`)
   - Renomme : `mv COORDINATION.<FOCUS>.md COORDINATION.<FOCUS>.bak.<timestamp>.md`
   - Crée un nouveau `COORDINATION.<FOCUS>.md` vide avec le template, mais **sans entrée join** (l'instance qui clear ne s'enregistre pas automatiquement) :
     ```markdown
     # COORDINATION.<FOCUS>.md

     > Protocole & légende : voir `COORDINATION.README.md`
     > Ce fichier coordonne les instances avec **focus = <focus>** uniquement.

     ---

     ## Contributeurs (focus: <focus>)

     | Instance | Joined | Notes |
     |---|---|---|
     | human | (toujours) | edits manuels ad hoc |

     **Lettres utilisées sur ce focus** : (aucune) → prochaine instance qui rejoint = `inst-A`

     ---

     ## Activité (ancien → récent, ajout en bas)

     ```

5. Confirme : *"Focus `<FOCUS>` archivé en `<bak filename>` et recréé vide. Pour participer, relance `/coordination` → 'Se connecter'."*

6. **Ne charge PAS le protocole** (cette instance n'est pas connectée). Termine le skill.

## Étape 7 — Application du protocole pour cette session

**Cette étape s'applique seulement si l'utilisateur a "Créé" ou "Connecté" (PAS si "Clear" ou "Ne rien faire").**

À partir de maintenant et pour le reste de la session, **TU DOIS appliquer ce protocole** :

> **Identité** : tu es `<inst-id>` sur focus `<FOCUS>`. Log = `<REPO_ROOT>/COORDINATION.<FOCUS>.md`.
>
> **Avant tout `Edit` ou `Write` sur un fichier de documentation** (PRD/, Sprints/, ou tout fichier .md de contenu) :
>
> 1. Lis le log `COORDINATION.<FOCUS>.md`
> 2. Vérifie qu'aucun `[LOCKED]` ou `[WAITING]` non périmé ne couvre ta zone (même fichier + section qui chevauche)
> 3. Si la zone est libre : ajoute une ligne `[LOCKED] <slug> by <inst-id> (claimed <HH:MM>, TTL 30min) — <fichier> §<section> — "<intention courte>"` à la fin du log
> 4. Fais ton `Edit`/`Write`
> 5. Modifie ta ligne en `[DONE] <slug> by <inst-id> (<HH:MM-début> → <HH:MM-fin>) — <fichier> §<section> — "<résultat court>"`
>
> **Si collision** (un autre `[LOCKED]` ou `[WAITING]` couvre ta zone) :
>
> 1. Ajoute une ligne `[WAITING] <slug> by <inst-id> (added <HH:MM>, waiting on <autre-inst>) — ...`
> 2. **Arrête** et demande à l'humain quoi faire : attendre, switcher de tâche, abandonner.
>
> **Si un lock que tu attendais devient `[DONE]`** :
>
> 1. RELIS le fichier modifié (en entier ou la section concernée)
> 2. Vérifie que ton plan initial tient toujours (peut-être que l'autre instance a tout bouleversé)
> 3. **Demande à l'humain "OK go ?"** avant de claim ton lock et d'écrire
>
> **Lecture, recherche, planification, réflexion : LIBRE.** Tu ne lockes JAMAIS pour ces actions.
>
> **Locks périmés** : si tu vois un `[LOCKED]` dont `claimed + TTL < maintenant`, ignore-le (il est périmé). Tu peux signaler à l'humain qu'il y a des locks zombies à nettoyer.

## Règles générales

- **Ne jamais committer automatiquement.** L'utilisateur décide quand committer.
- **Ne jamais supprimer un fichier sans confirmation explicite.**
- **Ne jamais dupliquer** une note dans CLAUDE.md ou une ligne dans .gitignore. Vérifie avant d'append.
- **Pas de duplication d'instances** : si l'utilisateur relance `/coordination` → "Se connecter" sur un focus où il est déjà inscrit, signale-le et demande s'il veut une 2e identité ou simplement réafficher l'état.
- **Format de slug** : kebab-case court, descriptif (ex: `bilan-9`, `user-story-3-2`). Pas de préfixe `<focus>:` car le fichier indique déjà le focus.
- **Tous les timestamps en HH:MM** (heure locale, format 24h). Pour les archives, format `YYYYMMDD-HHMMSS`.
- **Réponses en français.** Concis. Une phrase pour confirmer une action.
