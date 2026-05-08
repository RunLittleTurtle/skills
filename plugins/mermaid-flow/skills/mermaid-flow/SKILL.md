---
name: mermaid-flow
description: Transformer un flow (texte, fichier markdown, mermaid existant ou image) en un flowchart Mermaid SIMPLIFIÉ pour personnes peu techniques (max 10 étapes, idéalement 7). Analyse profonde du flow + validation interactive du plan d'analyse avant génération. Produit un fichier .md au format strict avec légende, palette pastel light-mode, emojis acteurs (👤 client, 🤖 IA, ⚙️ système, 🖥️ UI qui s'affiche, ⚖️ décision). À invoquer quand l'utilisateur veut vulgariser un processus métier en diagramme accessible.
---

# Mermaid Flow — vulgarisation de processus

Aide l'utilisateur à transformer n'importe quel flow (texte, fichier markdown, mermaid existant, image) en un diagramme Mermaid simplifié et lisible pour des personnes peu techniques. Format strict aligné sur la référence : 7 étapes idéales, légende intégrée, palette pastel light-mode, emojis acteurs.

**Réponds en français. Sois concis. Confirme les actions en une phrase.**

---

## Étape 1 — Récupérer l'input

### 1a. Identifier le type d'input

Si l'utilisateur n'a rien fourni d'explicite, demande-lui dans le chat (pas via AskUserQuestion) :

> *« Tu veux convertir quoi ? Du texte que tu colles ici, un fichier markdown local, un mermaid existant à simplifier, ou une image/screenshot ? »*

### 1b. Ingestion par type

**Texte/scénario dans le chat** : le contenu est déjà dans le contexte conversationnel. Ne lis rien de plus, passe à l'étape 2.

**Fichier markdown local** : demande le path absolu en texte libre. Utilise `Read` sur le fichier complet. Si le fichier contient déjà un bloc ```` ```mermaid ```` , traite-le comme une variante mermaid existant (ci-dessous) MAIS lis aussi le contexte texte autour (titres, paragraphes) pour enrichir l'analyse.

**Mermaid existant à simplifier** : peut être inline dans le chat OU dans un fichier. Si fichier, `Read`. Localise le bloc ```` ```mermaid ```` , parse les nodes (`A["..."]`, `B{"..."}`) et les liens (`-->`, `--|"label"|-->`, `~~~`). Identifie les opportunités de simplification :

- Nodes du même acteur consécutifs → candidats à fusion
- Nodes très techniques (« API call XYZ ») → reformuler en humain
- Plus de 10 nodes principaux → réduction obligatoire
- Pas d'emojis acteurs → ajouter selon classification (étape 2c)
- Pas de légende → ajouter le squelette LEG

Présente la **réduction proposée** dans le plan d'analyse de l'étape 3.

**Image/screenshot** : demande le path absolu de l'image. Utilise `Read` (multimodal) pour analyser visuellement. Identifie les éléments lisibles : nodes, flèches, labels, branches. Si l'image est ambiguë (texte flou, forme peu claire, screen partiel), demande à l'utilisateur de **décrire textuellement** le flow dans le chat. Ne pas inventer.

---

## Étape 2 — Analyse profonde du flow (interne)

Cette étape se fait **dans ta tête**, pas en sortie utilisateur. Réponds aux 5 questions ci-dessous avant de passer à l'étape 3.

### 2a. Persona cible

Pour qui ce flow existe-t-il ? Une **phrase humaine courte** qui parle d'un besoin réel, pas d'un rôle abstrait.

- Bon : *« Voyageur qui veut planifier un road-trip sans se prendre la tête. »*
- Mauvais : *« Utilisateur final du module de réservation. »*

Si pas explicite dans l'input, déduis du contexte : qui prend les décisions dans le flow ? C'est probablement la persona.

### 2b. Distinction

Qu'est-ce qui rend CE flow unique parmi les autres flows similaires ?

- Si plusieurs flows existent dans le contexte (ex: flow_1, flow_2, flow_3) : qu'est-ce que ce flow couvre que les autres ne couvrent pas ?
- Si un seul flow : quel moment-clé du processus métier ce flow capture ?

Cette distinction guide le **titre court** et le **blockquote contextuel** du fichier final.

### 2c. Acteurs et étapes brutes

Énumère mentalement TOUTES les étapes du flow (pas de filtre). Pour chacune, identifie l'acteur :

- 👤 **client** : verbe d'action humain (saisit, choisit, valide, lit, refuse, clique).
- 🤖 **IA** : analyse, génération, suggestion (analyse, propose, génère, recommande, traduit).
- ⚙️ **système** : job automatique invisible (sauvegarde, calcule, envoie, met à jour la BD, archive).
- 🖥️ **UI qui s'affiche** : écran, modal, fenêtre, panneau qui apparaît à l'utilisateur (« la fenêtre conversationnelle s'ouvre », « l'écran de résultats s'affiche », « un modal de confirmation apparaît »).
- ⚖️ **décision** : question conditionnelle fermée (losange).

**Distinction critique ⚙️ vs 🖥️** : le ⚙️ système est silencieux (backend invisible). Le 🖥️ UI est visible (quelque chose apparaît à l'écran). En cas de doute entre IA et système : si c'est un LLM ou de l'analyse contextuelle → 🤖. Si c'est un job automatique sans intelligence → ⚙️.

### 2d. Décisions et boucles

**Décisions** : cherche les mots-clés conditionnels dans l'input — `si`, `selon`, `dans le cas où`, `ou`, `sinon`. Une décision = un node losange `{"⚖️ Question ?"}` avec 2 branches (Oui/Non) ou N branches labellisées. Place la décision juste après l'étape qui produit l'information de décision. Le label de la décision est une **question fermée**.

**Boucles** : cherche les mots-clés `réessaie`, `recommence`, `refait`, `retour à l'étape`, `si pas OK alors`. Une boucle = un lien `D --> A` (retour vers une étape antérieure), souvent depuis la branche « Non » d'une décision. **Limite** : maximum 1 boucle par diagramme. Plus = vulgarisation ratée, fusionner ou simplifier.

### 2e. Réduction à 7 étapes (cap dur 10)

Règle stricte :

- **Si > 10 étapes brutes** : fusionne les étapes consécutives du **même acteur** en une seule. Ex : « IA analyse les dates » + « IA cherche les dispos » → « IA analyse les dates et cherche les dispos ».
- **Si toujours > 10** : élève le niveau d'abstraction (regrouper plusieurs micro-étapes sous un concept-parent : « configure le profil » au lieu de 5 sous-étapes).
- **Si < 5 étapes brutes** : NE PAS inventer des étapes. Garde tel quel et ajoute du contexte en parenthèses pour enrichir les labels.
- **Cible** : 7 étapes principales (hors décisions losanges, qui sont en plus).
- **Hard cap** : 10 nodes principaux maximum.

---

## Étape 3 — Proposer le plan et valider

D'abord, présente ton plan **en texte dans le chat** (pas dans la question, c'est plus lisible) :

> *« Plan d'analyse pour `<nom du flow>` :*
>
> *- **Persona** : <1 phrase>*
> *- **Distinction** : <1 phrase>*
> *- **N étapes identifiées** :*
>   *1. 👤 <étape 1>*
>   *2. 🖥️ <étape 2>*
>   *3. 🤖 <étape 3>*
>   *4. ⚖️ <décision>*
>   *5. 👤 <étape 5>*
>   *... (jusqu'à 7-10)*
> *- **Bifurcations** : <description ou « aucune »>*
> *- **Boucles** : <description ou « aucune »>*
> *- **Variante template** : <linéaire | avec décision | avec boucle | avec ref>*
> *- **Couleurs** : palette pastel light-mode standard (violet=client, bleu=IA, gris=système, vert=UI, jaune=décision) »*

Puis utilise `AskUserQuestion` :

- **Question** : *« OK pour générer le diagramme avec ce plan ? »*
- **Options** :
  - *« Oui, génère »* (Recommended) — procéder à la génération avec le plan ci-dessus
  - *« Ajuster »* — l'utilisateur précise via Other ce qu'il veut changer (persona, étapes, etc.)
  - *« Annuler »* — sortir du skill sans rien générer

**Si Ajuster** : prends le feedback, retravaille en interne, **reproposes** un nouveau plan avec un nouvel AskUserQuestion. Boucle jusqu'à validation explicite.

**Si Annuler** : termine le skill avec *« Génération annulée. »*

---

## Étape 4 — Générer le fichier markdown

### 4a. Choisir la variante de template

Selon la structure identifiée à l'étape 2 :

- **Linéaire simple** : pas de décision, pas de boucle.
- **Avec décision** : un ou plusieurs losanges avec branches Oui/Non.
- **Avec boucle** : lien retour vers un node précédent, souvent depuis branche Non d'une décision.
- **Avec référence à autre flow** : node `REF1["⚙️ Voir flow X"]` qui pointe vers un autre flow.

### 4b. Templates Mermaid

#### Squelette commun (toujours utilisé comme base)

````markdown
# Flow <X-S> - <titre court>

> <description du contexte en 1-2 phrases>
>
> **Persona : <persona en 1 phrase>**

```mermaid
%%{init: {'flowchart': {'curve': 'basis'}}}%%
flowchart TB
    subgraph LEG["Légende"]
        direction LR
        L1["👤 Action du client"] ~~~ L2["🤖 Action de l'assistant IA"] ~~~ L3["⚙️ Action du système"] ~~~ L4["🖥️ UI qui s'affiche"] ~~~ L5["⚖️ Décision"]
    end

    subgraph FLOW[" "]
        direction LR
        <NODES>

        <LIENS>
    end

    LEG ~~~ FLOW

    classDef clientAction fill:#E1BEE7,stroke:#6A1B9A,color:#000
    classDef iaAction fill:#BBDEFB,stroke:#1565C0,color:#000
    classDef systemAction fill:#CFD8DC,stroke:#455A64,color:#000
    classDef uiAction fill:#C8E6C9,stroke:#2E7D32,color:#000
    classDef decision fill:#FFF9C4,stroke:#F57F17,color:#000

    class <IDS_CLIENT> clientAction
    class <IDS_IA> iaAction
    class <IDS_SYSTEM> systemAction
    class <IDS_UI> uiAction
    class <IDS_DECISION> decision
    class L1 clientAction
    class L2 iaAction
    class L3 systemAction
    class L4 uiAction
    class L5 decision

    style FLOW fill:none,stroke:none
```
````

**Important** : la légende n'inclut un node qu'aux acteurs **réellement utilisés** dans le flow. Si pas d'UI : retirer L4, retirer la classDef `uiAction`, et renuméroter L5 décision en L4. Idem pour les autres acteurs absents.

#### Variante 1 — Linéaire simple (avec UI)

```
A["👤 1. Clique 'Générer avec IA'"]
B["🖥️ 2. Fenêtre conversationnelle s'ouvre"]
C["👤 3. Saisit ses critères (dates, voyageurs)"]
D["🤖 4. Analyse les besoins et propose 3 options"]
E["👤 5. Choisit une option"]
F["⚙️ 6. Sauvegarde dans le compte"]
G["🖥️ 7. Affiche le résumé final"]

A --> B --> C --> D --> E --> F --> G
```

#### Variante 2 — Avec décision

```
A["👤 1. Saisit sa demande"]
B["🤖 2. Analyse la demande"]
C{"⚖️ Demande complète ?"}
D["🤖 3. Génère la proposition"]
E["🤖 3'. Demande des précisions"]
F["👤 4. Valide ou ajuste"]
G["⚙️ 5. Sauvegarde"]

A --> B --> C
C -->|"Oui"| D
C -->|"Non"| E
E --> A
D --> F --> G
```

#### Variante 3 — Avec boucle

```
A["👤 1. Soumet une version"]
B["🤖 2. Analyse et donne feedback"]
C{"⚖️ Satisfait ?"}
D["👤 3. Ajuste"]
E["⚙️ 4. Sauvegarde la version finale"]

A --> B --> C
C -->|"Non"| D
D --> B
C -->|"Oui"| E
```

#### Variante 4 — Avec référence à un autre flow

```
A["👤 1. Lance la modification"]
REF1["⚙️ Voir flow 2-S — Création initiale"]
B["🤖 2. Récupère le contexte"]
C["👤 3. Sélectionne ce qui change"]
D["⚙️ 4. Met à jour"]

A --> REF1 --> B --> C --> D
```

Le node `REF1` est classé `systemAction` (gris) — pas de classDef séparée nécessaire.

### 4c. Assembler le fichier final

1. Choisis le squelette + la variante adaptée.
2. Remplis `<NODES>` avec les 7 étapes identifiées (IDs `A`, `B`, `C`, `D`, `E`, `F`, `G` en single letter ; losanges = lettre suivante).
3. Construis `<LIENS>` selon la structure (linéaire, branches Oui/Non, boucle, ref).
4. Remplis les listes `<IDS_CLIENT>`, `<IDS_IA>`, `<IDS_SYSTEM>`, `<IDS_UI>`, `<IDS_DECISION>` en classifiant chaque node selon son emoji.
5. Remplace le titre, le blockquote, la persona.
6. **Vérifie** que chaque node est classé dans exactement une classe (sinon Mermaid le rend gris par défaut).
7. **Vérifie** que les labels respectent : 7-10 mots max, parenthèses pour les détails, emoji au début.
8. **Vérifie** que la légende ne contient que les acteurs réellement présents dans le flux.

---

## Étape 5 — Sauvegarder et confirmer

### 5a. Déterminer l'emplacement

- Si l'input était un **fichier markdown** : propose de sauvegarder dans le **même dossier** que le fichier source, avec le suffixe `_legende.md` (ex: `flow_5.md` → `flow_5_legende.md`). Confirme via `AskUserQuestion`.
- Si l'input était **texte/mermaid/image** dans le chat : demande le path absolu via `AskUserQuestion`. Recommande `~/Desktop/flow_<nom>_legende.md`.

### 5b. Écrire et confirmer

1. `Write` du fichier markdown final au path validé.
2. Confirme en une phrase : *« Flowchart sauvegardé dans `<path>`. <N> étapes, variante <linéaire|décision|boucle|ref>. »*
3. **Une seule fois** (pas à chaque appel), suggère à l'utilisateur de prévisualiser avec un viewer Mermaid (mermaid.live, extension VS Code).

---

## Règles invariantes du format Mermaid

1. `flowchart TB` au niveau racine (empile LEG au-dessus de FLOW). `direction LR` dans CHAQUE subgraph (flux horizontal visuellement).
2. Toujours commencer par `%%{init: {'flowchart': {'curve': 'basis'}}}%%`.
3. Légende : subgraph `LEG["Légende"]`, direction LR, nodes L1-L5 connectés par `~~~` (pointillés invisibles).
4. Flux : subgraph `FLOW[" "]` (label = un espace), direction LR, style final `style FLOW fill:none,stroke:none`.
5. Connexion légende-flux : `LEG ~~~ FLOW`.
6. IDs : single letter `A`, `B`, `C`, ... pour les nodes principaux. `REF1`, `REF2`, ... pour les références à d'autres flows.
7. Actions = rectangles `["..."]`. Décisions = losanges `{"..."}`.
8. Emojis au début du label : `👤` (client), `🤖` (IA), `⚙️` (système, job invisible), `🖥️` (UI qui s'affiche : écran/modal/fenêtre visible), `⚖️` (décision).
9. Numérotation `"1. ..."` dans les labels : optionnelle, recommandée si > 5 étapes.
10. Labels de liens : `-->|"Oui"|` / `-->|"Non"|` pour décisions ; `-->` simple sinon ; `-- "label" -->` accepté pour multi-branches.
11. Concision : 7-10 mots max par label, parenthèses pour détails (ex: `"Saisit ses critères (dates, voyageurs)"`).
12. **Couleurs EXACTES (ne jamais modifier)** :
    - clientAction (👤) : `fill:#E1BEE7,stroke:#6A1B9A,color:#000` (violet pastel)
    - iaAction (🤖) : `fill:#BBDEFB,stroke:#1565C0,color:#000` (bleu pastel)
    - systemAction (⚙️) : `fill:#CFD8DC,stroke:#455A64,color:#000` (gris pastel)
    - uiAction (🖥️) : `fill:#C8E6C9,stroke:#2E7D32,color:#000` (vert pastel)
    - decision (⚖️) : `fill:#FFF9C4,stroke:#F57F17,color:#000` (jaune pastel)
13. Texte toujours `color:#000`.
14. Maximum 10 nodes principaux (hors légende), idéalement 7.
15. Chaque node DOIT être assigné à une classe (incluant les nodes de la légende L1-L5).
16. La légende n'inclut que les acteurs **réellement présents** dans le flow. Si pas d'UI : pas de node 🖥️ dans la légende et pas de classDef `uiAction`.

---

## Règles de comportement

- **Ne jamais générer le fichier sans validation explicite** de l'utilisateur (étape 3 obligatoire).
- **Ne jamais inventer** des étapes ou des acteurs si l'input est ambigu — demande à l'utilisateur.
- **Ne jamais modifier la palette de couleurs** — elle est imposée par la cohérence avec les flows existants.
- **Ne jamais dépasser 10 nodes principaux** (hors légende). Réduire ou découper en plusieurs flows.
- **Ne jamais commit automatiquement** le fichier généré. L'utilisateur décide.
- **Réponds en français.** Concis. Une phrase pour confirmer une action.
- Si l'utilisateur veut s'écarter du format strict (ex: « ajoute des couleurs vives »), **préviens** que ça casse la cohérence avec les autres flows existants, mais accepte si insistance.
