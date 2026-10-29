---
name: bulk-rnaseq-expression-preprocessing
description: Use when a bulk RNA-seq or microarray expression matrix for a cell population
  of interest must be reformatted (gene-symbol-indexed) before computing ICELLNET
  cell-cell communication scores against reference partner cells.
when_to_use_negative:
- Input is already single-cell data or a preformatted ICELLNET expression object —
  use the dedicated scRNA-seq tutorial/workflow instead of re-preprocessing
- The analysis goal is differential expression within one sample rather than inferring
  intercellular communication; ICELLNET preprocessing produces scoring inputs, not
  DE tables
- 'Non-human transcriptomes: the ICELLNET ligand-receptor database is curated exclusively
  from human studies'
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: ICELLNET
  role: Target framework whose expected gene-symbol-indexed expression format defines
    the preprocessing output; scores communication via a manually curated human ligand-receptor
    database accounting for ligand/receptor complex subunits
  repo: https://github.com/soumelis-lab/ICELLNET
- name: devtools
  role: Used to install the ICELLNET R package from GitHub after CRAN and Bioconductor
    dependencies are in place
- name: R
  role: Runtime environment for the preprocessing and ICELLNET scoring; version must
    match the ICELLNET release (v2.0.0 needs R < 4.1; v2.2.x for R > 4.1.x)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-21244-x
    title: Dissection of intercellular communication using the transcriptome-based
      framework ICELLNET
  source_content_hash: sha256:dd84e643fa2eb6f93c70e280a756fc2eeeeaaac7272d92511d16e5b1215c3f7d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/soumelis-lab/ICELLNET
  tools:
  - ICELLNET
  - R
  - devtools
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# bulk-rnaseq-expression-preprocessing

## Summary

Preprocess bulk RNA-seq expression matrices into the gene-symbol-indexed format required by the ICELLNET communication-scoring framework, so a cell population of interest (e.g. CAF-S1, n=6 and CAF-S4, n=3) can be connected to reference partner transcriptomes such as the Human Primary Cell Atlas.

## When to use

Use when a bulk RNA-seq or microarray expression matrix for a cell population of interest must be reformatted (gene-symbol-indexed) before computing ICELLNET cell-cell communication scores against reference partner cells.

## When NOT to use

- Input is already single-cell data or a preformatted ICELLNET expression object — use the dedicated scRNA-seq tutorial/workflow instead of re-preprocessing
- The analysis goal is differential expression within one sample rather than inferring intercellular communication; ICELLNET preprocessing produces scoring inputs, not DE tables
- Non-human transcriptomes: the ICELLNET ligand-receptor database is curated exclusively from human studies

## Inputs

- Bulk RNA-seq (or microarray) expression matrix for the cell population of interest (e.g. CAF-S1 n=6, CAF-S4 n=3 bulk RNA-seq matrices)
- Human Primary Cell Atlas reference transcriptome profiles (33 cell types) as partner-cell inputs
- Gene-symbol annotation mapping (rows indexed by gene symbols)

## Outputs

- Gene-symbol-indexed expression matrix formatted for ICELLNET scoring
- Partner-cell reference expression matrix ready for the ICELLNET communication-score computation
- Downstream: communication-score table (e.g. rescaled outward scores per partner cell, comparable across CAF-S1 and CAF-S4)

## How to apply

Install the ICELLNET dependencies with install.packages (devtools, jetset, readxl, psych, GGally, gplots, ggplot2, RColorBrewer, data.table, grid, gridExtra, ggthemes, scales, rlist) plus Bioconductor packages (BiocGenerics, org.Hs.eg.db, hgu133plus2.db, annotate), respecting the version constraint that ICELLNET v2.0.0 needs R < 4.1 while v2.2.x is required for R > 4.1.x. Install ICELLNET from GitHub via devtools::install_github('soumelis-lab/ICELLNET', ref='master', subdir='icellnet'). Load the bulk RNA-seq count/TPM matrices for the populations of interest and preprocess them so rows are gene symbols, the identifier expected by ICELLNET's expert-curated ligand-receptor database (1669 interactions in v2). Load the Human Primary Cell Atlas (reference transcriptomes of 33 cell types) as partner-cell profiles. The preprocessed matrices feed the ICELLNET scoring step, which quantifies communication scores and can then be rescaled per subset (e.g. CAF-S1 vs CAF-S4 outward scores) for cross-subset comparison.

## Related tools

- **ICELLNET** (Target framework whose expected gene-symbol-indexed expression format defines the preprocessing output; scores communication via a manually curated human ligand-receptor database accounting for ligand/receptor complex subunits) — https://github.com/soumelis-lab/ICELLNET
- **devtools** (Used to install the ICELLNET R package from GitHub after CRAN and Bioconductor dependencies are in place)
- **R** (Runtime environment for the preprocessing and ICELLNET scoring; version must match the ICELLNET release (v2.0.0 needs R < 4.1; v2.2.x for R > 4.1.x))

## Examples

```
library(devtools); install_github("soumelis-lab/ICELLNET", ref="master", subdir="icellnet")
```

## Evaluation signals

- Expression matrix rows are gene symbols (the identifier space of the ICELLNET ligand-receptor database) before scoring
- Installed ICELLNET version matches the R version: v2.0.0 only under R < 4.1, v2.2.x under R > 4.1.x
- Communication scores are computed for the population of interest against each of the Primary Cell Atlas partner cell types
- Outward scores rescaled per CAF subset reproduce the expected ordering, e.g. CAF-S1 > CAF-S4 for epithelial (6 vs 3), endothelial (7 vs 4), pDC (6 vs 4), and B cells (3 vs 1)
- Output score table written with per-partner comparisons (e.g. caf_s1_s4_icellnet_scores.csv) contains rescaled values comparable across subsets

## Limitations

- ICELLNET v2.0.0 requires R version < 4.1; on R > 4.1.x you must switch to ICELLNET v2.2.x or installation/analysis fails
- The ICELLNET database is human-specific and, although extended to 1669 interactions in v2, is relatively small compared to other databases and excludes chemokine and checkpoint interactions
- The database must be paired with correctly symbol-mapped genes; multimeric ligand/receptor complexes rely on co-expression rules for protein subunits, so subunits must not be dropped during preprocessing

## Evidence

- [methods] install_github("soumelis-lab/ICELLNET",ref="master", subdir="icellnet"): "install_github("soumelis-lab/ICELLNET",ref="master", subdir="icellnet")"
- [methods] with the Primary Cell Atlas (Database of reference transcriptome of 33 cell types): "with the Primary Cell Atlas (Database of reference transcriptome of 33 cell types)"
- [readme] ICELLNET v2.0.0 requires R version < 4.1. For latest R versions (> 4.1.x), please use ICELLNET v2.2.x.: "ICELLNET v2.0.0 requires R version < 4.1. For latest R versions (> 4.1.x), please use ICELLNET v2.2.x."
- [readme] Versatile tool applicable on a wide range of transcriptomic technologies (microarray, bulk RNAseq, scRNAseq, spatial transcriptomics): "Versatile tool applicable on a wide range of transcriptomic technologies (microarray, bulk RNAseq, scRNAseq, spatial transcriptomics)"
- [discussion] manually curated ligand-receptor interactions database: curated exclusively from human studies: "With ICELLNET, we have developed a fully manually curated database, combining biological relevance, ease of use, and experimental validation."
