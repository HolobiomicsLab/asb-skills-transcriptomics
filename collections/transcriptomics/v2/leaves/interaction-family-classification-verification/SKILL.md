---
name: interaction-family-classification-verification
description: Use when building or auditing a cell-cell communication database and
  you must confirm that curated ligand-receptor interactions are complete, counted
  correctly, and classified into the expected molecular families before running inference.
when_to_use_negative:
- The input database is already a scored communication matrix rather than a raw curated
  interaction list — validation counts would be meaningless
- You are using ICELLNET v2 with the extended database (1669 interactions); the expected
  counts of 380 interactions / 194 cytokine-receptor interactions apply only to the
  original curated database from Supplementary Data 1
- You need de novo prediction of ligand-receptor pairs; this skill only verifies existing
  expert curation and family assignment
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_0128
tools:
- name: ICELLNET
  role: Source of the expert-curated ligand-receptor database (Supplementary Data
    1) that is loaded and validated; the framework whose interaction families are
    being checked
  repo: https://github.com/soumelis-lab/ICELLNET
- name: R
  role: Runtime environment for loading, counting, grouping, and writing the validation
    report
- name: readxl
  role: R package used to read the Excel-format Supplementary Data 1 interaction table
- name: devtools
  role: Used to install the ICELLNET R package from GitHub before accessing its data
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-21244-x
    title: Dissection of intercellular communication using the transcriptome-based
      framework ICELLNET
  source_content_hash: sha256:41cc844702cc7b7a77d93d6838dcbc9da6bed32c78947bc396418c7561ca25b6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/soumelis-lab/ICELLNET
  tools:
  - ICELLNET
  - R
  - devtools
  - readxl
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_0128
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# interaction-family-classification-verification

## Summary

Validates an expert-curated ligand-receptor interaction database (e.g. the ICELLNET database in Supplementary Data 1) by loading it in R, counting curated interactions, and verifying that every interaction is assigned to one of the defined molecular families (Growth factors, Cytokines, Chemokines, Immune Checkpoints, Notch signaling, Antigen binding/other).

## When to use

Use when building or auditing a cell-cell communication database and you must confirm that curated ligand-receptor interactions are complete, counted correctly, and classified into the expected molecular families before running inference.

## When NOT to use

- The input database is already a scored communication matrix rather than a raw curated interaction list — validation counts would be meaningless
- You are using ICELLNET v2 with the extended database (1669 interactions); the expected counts of 380 interactions / 194 cytokine-receptor interactions apply only to the original curated database from Supplementary Data 1
- You need de novo prediction of ligand-receptor pairs; this skill only verifies existing expert curation and family assignment

## Inputs

- ICELLNET expert-curated ligand-receptor interactions table (Supplementary Data 1, Excel/delimited format from soumelis-lab/ICELLNET)
- Expected reference counts (e.g. 380 total interactions, 6 families, 194 cytokine-receptor interactions across 7 subfamilies)

## Outputs

- lr_database_validation_report.csv containing observed vs expected counts per family and per-family match flags (all TRUE when validation passes)
- Per-family count table of classified interactions (Growth factors, Cytokines, Chemokines, Immune Checkpoints, Notch signaling, Antigen binding/other)

## How to apply

Load the curated interaction table (Supplementary Data 1 of the ICELLNET paper) with readxl or data.table in R. Count total ligand-receptor interaction rows, keeping in mind that each interaction row accounts for the multiple subunits of ligand and receptor complexes (as in CellPhoneDB-style logical subunit rules). Group interactions by family and confirm presence of the 6 major families: Growth factors, Cytokines, Chemokines, Immune Checkpoints, Notch signaling, and Antigen binding/other. For the cytokine family, further verify the distinct subfamilies (expected 7) and the total cytokine-receptor interactions (expected 194). Assemble observed vs expected counts per family into a summary table and write it with write.csv as lr_database_validation_report.csv, flagging each observed-vs-expected comparison as TRUE/FALSE.

## Related tools

- **ICELLNET** (Source of the expert-curated ligand-receptor database (Supplementary Data 1) that is loaded and validated; the framework whose interaction families are being checked) — https://github.com/soumelis-lab/ICELLNET
- **R** (Runtime environment for loading, counting, grouping, and writing the validation report)
- **readxl** (R package used to read the Excel-format Supplementary Data 1 interaction table)
- **devtools** (Used to install the ICELLNET R package from GitHub before accessing its data)

## Examples

```
library(readxl); lr <- read_excel("Supplementary_Data_1.xlsx"); nrow(lr); table(lr$family); write.csv(data.frame(family=names(table(lr$family)), observed=as.integer(table(lr$family))), "lr_database_validation_report.csv")
```

## Evaluation signals

- Observed total interaction count equals the expected 380 curated ligand-receptor interactions
- All 6 expected families are present: Growth factors, Cytokines, Chemokines, Immune Checkpoints, Notch signaling, Antigen binding/other
- Cytokine family contains 194 cytokine-receptor interactions distributed across exactly 7 distinct subfamilies
- lr_database_validation_report.csv exists and every observed-vs-expected comparison flag is TRUE
- No interaction row is left unassigned to a family after grouping

## Limitations

- The original curated database contains a relatively small number of interactions compared to other databases, and chemokine and checkpoint interactions were not fully included at publication time; ICELLNET v2 extends the database to 1669 interactions, so version-specific expected counts must be used
- The database is curated exclusively from human studies and does not integrate downstream effectors; some receptors (DR6/TNFRSF21, RELT, TROY, NGFR) and ligands (IL17D) lack known partners
- ICELLNET v2.0.0 requires R version < 4.1; for R > 4.1.x use ICELLNET v2.2.x

## Evidence

- [abstract] expert-curated database of ligand-receptor interactions: [evidence span withheld — anchor status: not_found]
- [discussion] multiple subunits handled: "Similar to CellPhoneDB10, ICELLNET takes into account the multiple subunits of ligands and receptors, by introducing logical rules for co-expression of protein subunits."
- [discussion] family-specific curation specificity: "Although ICELLNET includes a relatively small number of interactions compared to other existing databases, it is very specific and exhaustive for cytokine interactions, and will in time be extended"
- [methods] dependency installation: "install.packages(c("devtools", "jetset", "readxl", "psych", "GGally", "gplots", "ggplot2", "RColorBrewer", "data.table", "grid", "gridExtra", "ggthemes", "scales","rlist"))"
- [readme] database extension in v2: "New structure and extension of ICELLNET database up to 1669 interactions."
