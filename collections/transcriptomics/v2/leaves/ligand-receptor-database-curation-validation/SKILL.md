---
name: ligand-receptor-database-curation-validation
description: Use when you need to load, audit, or validate a manually curated ligand–receptor
  interaction database (e.g. ICELLNET Supplementary Data 1) before using it for cell–cell
  communication scoring, or when reproducing its family-level and subfamily-level
  interaction counts.
when_to_use_negative:
- You are using ICELLNET v2 (1669 interactions) rather than the v1 database of the
  2021 paper — expected counts of 380/194 will fail and the database has a new structure
- You want to score communication between transcriptomic samples — use the ICELLNET
  communication-score quantification workflow instead, which consumes this validated
  database as input
- You need a non-human, non-expert-curated interaction set — ICELLNET's database is
  curated exclusively from human studies and is deliberately smaller than other databases
  (e.g. CellPhoneDB)
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_0602
tools:
- name: ICELLNET
  role: Provides the expert-curated ligand–receptor database being validated and the
    R environment in which it is loaded and audited
  repo: https://github.com/soumelis-lab/ICELLNET
- name: readxl
  role: Reads the Supplementary Data 1 ligand–receptor interaction table (Excel format)
    into R for counting
- name: data.table
  role: Alternative fast reader for delimited interaction-table formats during loading
    and grouping
- name: devtools
  role: Installs the ICELLNET R package from GitHub so the bundled database and metadata
    are available
  repo: https://github.com/soumelis-lab/ICELLNET
- name: R
  role: Runtime for loading, filtering, counting, and writing the validation report;
    ICELLNET v2.0.0 requires R < 4.1
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-21244-x
    title: Dissection of intercellular communication using the transcriptome-based
      framework ICELLNET
  source_content_hash: sha256:88c84d7a9daf6e6e7a0586884866573f5ab7610614bc24dbc67fc826fb797094
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/soumelis-lab/ICELLNET
  tools:
  - ICELLNET
  - R
  - data.table
  - devtools
  - readxl
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# ligand-receptor-database-curation-validation

## Summary

Reproduce and validate the ICELLNET expert-curated ligand–receptor interaction database (380 curated interactions across 6 families: Growth factors, Cytokines, Chemokines, Immune Checkpoints, Notch signaling, Antigen binding/other), including the 194 cytokine-receptor interactions across 7 subfamilies, by loading Supplementary Data 1 from the soumelis-lab/ICELLNET repository and comparing observed counts against expected values.

## When to use

Use when you need to load, audit, or validate a manually curated ligand–receptor interaction database (e.g. ICELLNET Supplementary Data 1) before using it for cell–cell communication scoring, or when reproducing its family-level and subfamily-level interaction counts.

## When NOT to use

- You are using ICELLNET v2 (1669 interactions) rather than the v1 database of the 2021 paper — expected counts of 380/194 will fail and the database has a new structure
- You want to score communication between transcriptomic samples — use the ICELLNET communication-score quantification workflow instead, which consumes this validated database as input
- You need a non-human, non-expert-curated interaction set — ICELLNET's database is curated exclusively from human studies and is deliberately smaller than other databases (e.g. CellPhoneDB)

## Inputs

- ICELLNET ligand–receptor interactions database (Supplementary Data 1 from soumelis-lab/ICELLNET, .xlsx or delimited text)
- Expected count manifest (380 total interactions, 6 families, 194 cytokine-receptor interactions across 7 subfamilies)

## Outputs

- lr_database_validation_report.csv — observed-vs-expected counts per family with all TRUE validation flags
- Per-family and per-subfamily interaction count tables (e.g. 194 cytokine interactions across 7 subfamilies)

## How to apply

Load the ICELLNET ligand-receptor interactions table (Supplementary Data 1) from the soumelis-lab/ICELLNET repository into R using readxl (for Excel) or data.table (for delimited formats). Count total curated interaction rows — note that, like CellPhoneDB, ICELLNET accounts for the multiple subunits of ligand and receptor complexes, so each row may represent multimeric partners; the expected total is 380 interactions for the v1 database described in the 2021 paper (ICELLNET v2 extends this to 1669 interactions, so confirm the version first). Group interactions by family and confirm all 6 major families are present: Growth factors, Cytokines, Chemokines, Immune Checkpoints, Notch signaling, and Antigen binding/other. Filter to the cytokine family and verify 194 cytokine-receptor interactions spanning 7 distinct cytokine subfamilies. Assemble observed vs. expected counts per family into a summary table and write it with write.csv as lr_database_validation_report.csv; the validation passes only when observed total (380), family count (6), and cytokine counts (194 across 7 subfamilies) each match expected values (all comparison flags TRUE).

## Related tools

- **ICELLNET** (Provides the expert-curated ligand–receptor database being validated and the R environment in which it is loaded and audited) — https://github.com/soumelis-lab/ICELLNET
- **readxl** (Reads the Supplementary Data 1 ligand–receptor interaction table (Excel format) into R for counting)
- **data.table** (Alternative fast reader for delimited interaction-table formats during loading and grouping)
- **devtools** (Installs the ICELLNET R package from GitHub so the bundled database and metadata are available) — https://github.com/soumelis-lab/ICELLNET
- **R** (Runtime for loading, filtering, counting, and writing the validation report; ICELLNET v2.0.0 requires R < 4.1)

## Examples

```
library(readxl); lr <- read_excel('SupplementaryData1_LR_database.xlsx'); nrow(lr); table(lr$family)
```

## Evaluation signals

- Total curated interaction rows equal 380 (v1 database), accounting for multiple subunits of ligand and receptor complexes
- Grouping by family yields exactly 6 families: Growth factors, Cytokines, Chemokines, Immune Checkpoints, Notch signaling, Antigen binding/other
- Cytokine family filter yields 194 cytokine-receptor interactions across 7 distinct subfamilies
- All observed-vs-expected comparison flags in lr_database_validation_report.csv are TRUE
- Install path matches the documented install_github('soumelis-lab/ICELLNET', ref='master', subdir='icellnet') and R version constraint (< 4.1 for v2.0.0)

## Limitations

- ICELLNET v1 (380 interactions) is deliberately smaller than other databases such as CellPhoneDB; chemokine and immune-checkpoint coverage was still being extended at publication time
- Database is curated exclusively from human studies — interactions cannot be assumed for other organisms
- Version mismatch is a common failure mode: ICELLNET v2 extended the database to 1669 interactions, so 380/194 expected counts only apply to the 2021 Supplementary Data 1
- Some receptors lack known ligands (DR6/TNFRSF21, RELT, TROY, NGFR) and some ligands lack known receptors (e.g. IL17D), so family counts may include orphan entries

## Evidence

- [abstract] original expert-curated database of ligand-receptor interactions: "We develop ICELLNET, a transcriptomic-based framework integrating: 1) an original expert-curated database of ligand-receptor interactions"
- [methods] count equals 380: [evidence span withheld — anchor status: not_found]
- [methods] group by 6 families: [evidence span withheld — anchor status: not_found]
- [methods] cytokine filter counts: [evidence span withheld — anchor status: not_found]
- [discussion] subunit-aware curation: "Similar to CellPhoneDB10, ICELLNET takes into account the multiple subunits of ligands and receptors, by introducing logical rules for co-expression of protein subunits."
- [readme] v2 database extension: "New structure and extension of ICELLNET database up to 1669 interactions."
- [readme] human-only curation scope: "curated exclusively from **human** studies, ICELLNET database takes into account the **multiple** subunits of ligand and receptor complexes"
- [discussion] coverage limitation: "Although ICELLNET includes a relatively small number of interactions compared to other existing databases, it is very specific and exhaustive for cytokine interactions"
