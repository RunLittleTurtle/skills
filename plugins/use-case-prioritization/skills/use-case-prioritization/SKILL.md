---
name: use-case-prioritization
description: Score, prioritize, and build business cases for AI/automation use cases from any process discovery input (post-it photos, atelier transcripts, process inventories of up to 500+ rows, meeting notes, or mixed workshop materials). Combines BABOK Business Case methodology, UiPath Suitability scoring, fully-loaded cost calculations, and confidence-weighted ROI with verdict thresholds. Use this skill whenever the user has process discovery materials and needs to decide which use case to scope first, build an enveloppe budgétaire, or triage an automation pipeline. Triggers on keywords like prioriser, prioritization, use case, ROI, payback, business case, suitability, scoping, enveloppe budgétaire, quel use case en premier, automation pipeline, process triage, atelier découverte, post-its, cartographie processus, chiffrer use case. Always outputs both a working CSV (43-column v4 schema with FORMULES and SOURCE rows) and an executive summary markdown in French Quebec.
---

# Use Case Prioritization Framework

This skill turns process discovery materials into a structured priorisation artifact for Product Managers and Functional Analysts who need to decide which AI/automation use case to scope first.

## What this skill produces

Two files, always:

1. **`priorisation_use_cases.csv`** (43 columns, French Quebec headers): the working file for the analyst. Includes FORMULES row (row 2) and SOURCE row (row 3) under the header for traceability.

2. **`synthese_priorisation.md`**: executive summary for directeurs de compte and stakeholders. Top 5 priorities with 2-3 lines each, plus "À éliminer" and "À retravailler" sections.

## Always start by reading the framework

Before processing any input, read `references/framework_v4.md` in full. It explains the 43 columns, their sources (BABOK, UiPath, SAFe, DAMA-DMBOK, etc.), and the calculation logic. Without this context, you cannot fill the CSV correctly.

Then read `references/calculation_rules.md` for the exact formulas, default values, and verdict thresholds.

## Inputs this skill accepts

- **Photos of post-its / whiteboards** from ateliers de découverte
- **Transcripts** of meetings with stakeholders or architects (any language, but output in French Quebec)
- **Process inventory tables** (Excel/CSV with up to 500+ processes)
- **Free-form notes** from workshops
- **Mixed inputs** (combination of any of the above)

If no input is provided, ask the user what materials to work from. Do not invent use cases.

## Workflow

### Step 1: Inventory the use cases

For each input source:
- A use case is a **centered concept** (process, task, workflow) with associated pain points, volumes, stakeholders, and impact indicators
- In post-it diagrams, use cases are typically the central blue/dominant notes with arrows pointing in (causes) and out (pain points or consequences)
- In transcripts, use cases are the topics being discussed for automation
- In tables, each row is typically a use case (verify column meaning first)

Create one row per use case. Do not duplicate. If two sources describe the same process, merge their information into one row.

### Step 2: Extract what is stated, leave the rest empty

For each use case, fill columns based on what the input **explicitly states or strongly implies**. Never fabricate quantitative data. Empty cells are valuable: they correctly lower the Confiance Données score and signal what needs to be validated.

**Qualitative scores (Suitability 1-5)** can be estimated from context. If the input mentions "errors every day" and "20 people involved", you can reasonably estimate Impact Erreurs = 4 and Volume Standardisation = 4. Document the reasoning in the Notes column.

**Quantitative inputs (volumes, hours, costs)** must come from the input. If the transcript says "30h/semaine", use 30. If the input has no time data, leave Temps Actuel empty.

### Step 3: Apply the formulas

Once inputs are extracted, calculate the derived columns using formulas from `references/calculation_rules.md`:
- Suitability Score (col 14)
- Gain Heures Annuel (col 23)
- Bénéfice Brut Annuel (col 24)
- Bénéfice Net Annuel (col 26)
- Build Base via T-shirt lookup (col 28)
- Run Annuel default = Build × 15% if empty (col 30)
- Coût An 1 Total (col 31)
- Payback (col 32)
- ROI An 1 (col 33)
- Score Priorité (col 35)
- Nb Colonnes Critiques Remplies (col 36, counted over 20 critical columns)
- Confiance Données (col 39)
- Confiance Globale (col 40)
- Verdict Confiance (col 41)
- Verdict ROI (col 42)

### Step 4: Build the CSV

Use `references/csv_template.csv` as the starting structure. It contains:
- Row 1: Header (43 columns)
- Row 2: FORMULES (do not modify)
- Row 3: SOURCE (do not modify)
- Then append data rows (one per use case)

Save to `/mnt/user-data/outputs/priorisation_use_cases.csv`.

### Step 5: Generate the executive summary

Use `references/exec_summary_template.md` for the format. Key rules:
- Maximum 5 use cases in the "Top priorités" section
- 2-3 lines per use case
- Include "À éliminer" section (Verdict ROI = Pass)
- Include "À retravailler" section (Verdict Confiance < 60%)
- French Quebec, no em dashes (use commas, parentheses, or colons instead)

Save to `/mnt/user-data/outputs/synthese_priorisation.md`.

### Step 6: Present both files

Use the `present_files` tool to share both outputs. The CSV first (primary working artifact), then the markdown summary.

## Critical principles

**Never fabricate data**. Empty cells lower confidence, which is correct behavior. A CSV with 60% confidence is honest and useful. A CSV with 95% confidence built on invented numbers is worse than useless because it misleads decision-making.

**Use "N/A" sparingly**. N/A means "the question genuinely doesn't apply to this use case", not "I don't know". N/A counts as a filled cell for confidence. Empty counts as not filled.

**Don't override verdicts**. Verdicts are calculated from thresholds. If the math says Pass, don't write Go because you have a hunch. Hunches go in the Notes column.

**The architect output is just T-shirt size**. If no architect input is available, leave Taille Build empty (it is a critical column, so confidence will drop). Note in the Notes column that architect input is needed.

**Output language is always French Quebec**. Even if the input is in English, translate. Never use em dashes in any output text (replace with comma, parenthesis, or colon).

## Edge cases

- **No quantitative data at all**: Build the CSV with qualitative columns filled. Confiance Données will be low (30-50%). The synthèse explicitly recommends a chiffrage workshop. This is the correct outcome, not a failure.

- **Single process**: Generate a 1-row CSV. The synthèse focuses on that single use case with deeper analysis.

- **500+ processes table**: Process all rows. The CSV handles it. The synthèse still highlights top 5. If processing is too long, batch by department or by Suitability Score and warn the user.

- **Conflicting information between sources**: Document the conflict in Notes column. Use the most credible source for the cell value (transcript validated > note d'atelier > photo de post-it).

- **Architect input arrives later**: User can update the CSV themselves and the Confiance Globale recalculates. The skill handles initial extraction, not ongoing updates.

## Quality checks before presenting

Before calling present_files, verify:
- [ ] CSV has FORMULES row and SOURCE row preserved
- [ ] Each data row has at least Use Case and Pain Point filled
- [ ] Verdict ROI matches Payback threshold (see calculation_rules.md)
- [ ] Verdict Confiance matches Confiance Globale threshold
- [ ] No em dashes anywhere in either file
- [ ] Synthèse has top priorités, à éliminer, à retravailler sections
- [ ] Notes column flags major hypotheses or missing data points
