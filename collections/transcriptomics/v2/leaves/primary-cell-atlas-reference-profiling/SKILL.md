---
name: primary-cell-atlas-reference-profiling
description: Use when you have a gene-symbol-indexed transcriptome of one cell population
  of interest and need to infer its emitted (outward) communication toward reference
  partner cell types via ICELLNET.
when_to_use_negative:
- The input is already a computed cell-cell communication score table rather than
  a raw transcriptome of a single cell type of interest
- The analysis involves non-human transcriptomic data — the ICELLNET database is curated
  exclusively from human studies
- You need chemokine or immune-checkpoint interactions, which are not yet included
  in the ICELLNET database
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: ICELLNET
  role: Computes communication scores between the cell population of interest and
    Primary Cell Atlas reference partner profiles using its curated ligand-receptor
    database
  repo: https://github.com/soumelis-lab/ICELLNET
- name: devtools
  role: Installs the icellnet package from GitHub via install_github("soumelis-lab/ICELLNET",
    ref="master", subdir="icellnet")
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-21244-x
    title: Dissection of intercellular communication using the transcriptome-based
      framework ICELLNET
  source_content_hash: sha256:0a57dddc07e2e5bd103e936e1a9bbd6ecc956a016437d8344fae00fde3e9688c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/soumelis-lab/ICELLNET
  tools:
  - ICELLNET
  - R
  - devtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# primary-cell-atlas-reference-profiling

## Summary

Profile a single cell type of interest (e.g., CAF-S1 vs CAF-S4 bulk RNA-seq) for outward cell-cell communication by connecting its transcriptome to the Human Primary Cell Atlas reference profiles using the ICELLNET scoring framework.

## When to use

Use when you have a gene-symbol-indexed transcriptome of one cell population of interest and need to infer its emitted (outward) communication toward reference partner cell types via ICELLNET.

## When NOT to use

- The input is already a computed cell-cell communication score table rather than a raw transcriptome of a single cell type of interest
- The analysis involves non-human transcriptomic data — the ICELLNET database is curated exclusively from human studies
- You need chemokine or immune-checkpoint interactions, which are not yet included in the ICELLNET database

## Inputs

- Gene-symbol-indexed bulk RNA-seq or microarray expression matrix for the cell population of interest (e.g., CAF-S1 n=6 and CAF-S4 n=3)
- Human Primary Cell Atlas reference transcriptome profiles (database of reference transcriptomes of 33 cell types) as partner cell input
- ICELLNET expert-curated human ligand-receptor interactions database (1669 interactions in v2)

## Outputs

- Communication score matrix linking the cell population of interest to each Primary Cell Atlas partner cell type
- Rescaled outward (emitted) communication scores per cell subset, with partner cells ranked by score (e.g., written to caf_s1_s4_icellnet_scores.csv)

## How to apply

Install ICELLNET with its CRAN dependencies (devtools, jetset, readxl, psych, GGally, gplots, ggplot2, RColorBrewer, data.table, grid, gridExtra, ggthemes, scales, rlist) and Bioconductor dependencies (BiocGenerics, org.Hs.eg.db, hgu133plus2.db, annotate), respecting that ICELLNET v2.0.0 requires R < 4.1 while v2.2.x is needed for R > 4.1.x. Preprocess the bulk expression matrices (CAF-S1 n=6, CAF-S4 n=3) into gene-symbol-indexed format, then load the Human Primary Cell Atlas (reference transcriptomes of 33 cell types) as partner profiles. Run the ICELLNET scoring step with the expert-curated human ligand-receptor database (1669 interactions in v2), which enforces co-expression rules for multiple subunits of ligand and receptor complexes. Rescale the outward (emitted) scores per subset and rank partner cells; for the CAF case, verify CAF-S1 > CAF-S4 toward epithelial (6 vs 3), endothelial (7 vs 4), pDC (6 vs 4), and B cells (3 vs 1).

## Related tools

- **ICELLNET** (Computes communication scores between the cell population of interest and Primary Cell Atlas reference partner profiles using its curated ligand-receptor database) — https://github.com/soumelis-lab/ICELLNET
- **devtools** (Installs the icellnet package from GitHub via install_github("soumelis-lab/ICELLNET", ref="master", subdir="icellnet"))

## Examples

```
library(devtools); install_github("soumelis-lab/ICELLNET", ref="master", subdir="icellnet")
```

## Evaluation signals

- Rescaled outward scores reproduce the reported CAF-S1 > CAF-S4 pattern: epithelial 6 vs 3, endothelial 7 vs 4, pDC 6 vs 4, B cells 3 vs 1
- Scores are computed against the full set of Primary Cell Atlas reference transcriptomes (33 cell types), not an ad hoc partner list
- The scoring run uses the expert-curated database accounting for multiple subunits of ligand and receptor complexes (1669 interactions in v2)
- The R/ICELLNET version combination is valid (v2.0.0 with R < 4.1, or v2.2.x with R > 4.1.x)

## Limitations

- ICELLNET v2.0.0 requires R version < 4.1; R > 4.1.x requires ICELLNET v2.2.x
- ICELLNET includes a relatively small number of interactions compared to other databases; chemokine and checkpoint interactions are not yet included
- The database is curated exclusively from human studies, so cross-species application is unsupported

## Evidence

- [abstract] Connect single cell type to Primary Cell Atlas partners: "the possibility to connect a cell population of interest with 31 reference human cell types"
- [readme] CAF tutorial with Primary Cell Atlas: "infer cell-cell communication from the transcriptome of a single cell type of interest with the Primary Cell Atlas (Database of reference transcriptome of 33 cell types)."
- [discussion] HPCA as reference for partner integration: "ICELLNET allows us to integrate cell communication partners (sender or receiver) not included in a given"
- [readme] Subunit-aware curated database: "curated exclusively from human studies, ICELLNET database takes into account the multiple subunits of ligand and receptor complexes (more than 1600 interactions)"
- [methods] Database size in v2: "New structure and extension of ICELLNET database up to 1669 interactions."
