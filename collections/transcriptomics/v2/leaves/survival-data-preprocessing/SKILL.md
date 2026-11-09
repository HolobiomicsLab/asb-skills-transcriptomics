---
name: survival-data-preprocessing
description: Use when bulk RNA-seq or microarray expression and matched survival data
  must be cleaned and normalized before building marker gene-pair (IMGP) prognostic
  models, e.g., TCGA SKCM06 melanoma cohort input.
when_to_use_negative:
- Input is already a normalized, filtered expression matrix with validated survival
  endpoints — re-running TimiCheckEvent/TimiPrePropress could double-transform the
  data
- Analysis goal is scRNA-seq deconvolution or cell-type abundance estimation with
  tools like xCell or CIBERSORTx rather than gene-pair-based prognostic modeling
- The cohort lacks matched survival information; TimiGP requires a large expression
  dataset with survival data to identify prognostic gene pairs
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: TimiGP
  role: R package providing TimiCheckEvent and TimiPrePropress for clinical filtering
    and marker-expression preprocessing
  repo: https://github.com/CSkylarL/TimiGP
- name: R
  role: Runtime environment; TimiGP installed via devtools::install_github('CSkylarL/TimiGP')
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.xcrm.2023.101121
    title: 'TimiGP: Inferring cell-cell interactions and prognostic associations in
      the tumor immune microenvironment through gene pairs'
  source_content_hash: sha256:624f63cb5329c4be13ce44d04bee5d5ecc9c0d0afaa0b31a8423c46a37126c7c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/CSkylarL/TimiGP
  tools:
  - R
  - TimiGP
  - survival
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# survival-data-preprocessing

## Summary

Quality-filter and normalize clinical survival information and bulk transcriptome profiles prior to TimiGP marker-pair analysis, using TimiCheckEvent to drop low-quality event/time-to-event records and TimiPrePropress to extract marker expression with log transformation and gene-wise median normalization.

## When to use

Use when bulk RNA-seq or microarray expression and matched survival data must be cleaned and normalized before building marker gene-pair (IMGP) prognostic models, e.g., TCGA SKCM06 melanoma cohort input.

## When NOT to use

- Input is already a normalized, filtered expression matrix with validated survival endpoints — re-running TimiCheckEvent/TimiPrePropress could double-transform the data
- Analysis goal is scRNA-seq deconvolution or cell-type abundance estimation with tools like xCell or CIBERSORTx rather than gene-pair-based prognostic modeling
- The cohort lacks matched survival information; TimiGP requires a large expression dataset with survival data to identify prognostic gene pairs

## Inputs

- TCGA SKCM06 (metastatic melanoma) clinical info (SKCM06info) with event and time-to-event fields
- TCGA SKCM06 transcriptome profile (SKCM06rna), RNA-seq or microarray
- Cell-type marker annotation, e.g., CellType_Charoentong2017_Bindea2013_Xu2018_Immune

## Outputs

- Filtered clinical survival table with valid event/time-to-event records
- Log-transformed, gene-wise median-normalized marker expression matrix for cohort samples
- Preprocessed data objects ready for TimiGenePair marker-pair score (MPS) generation

## How to apply

First pass the clinical info (event and time-to-event columns) through TimiCheckEvent, which filters out low-quality or malformed survival records. Then run TimiPrePropress with log = TRUE and GMNorm = TRUE on the transcriptome profile and the chosen marker annotation (e.g., CellType_Charoentong2017_Bindea2013_Xu2018_Immune); this extracts only the selected markers' rows for the cohort samples, log-transforms expression, and applies gene-wise median normalization. This step precedes TimiGenePair, so downstream pairwise comparisons and TimiCOX univariate regression assume normalized, log-scale marker expression paired with valid survival endpoints. In the TimiGP manuscript workflow this is Step 1 ('Preprocess of clinical information and transcriptome profile') of the five-step framework, applied to inputs such as SKCM06info (clinical) and SKCM06rna (transcriptome).

## Related tools

- **TimiGP** (R package providing TimiCheckEvent and TimiPrePropress for clinical filtering and marker-expression preprocessing) — https://github.com/CSkylarL/TimiGP
- **R** (Runtime environment; TimiGP installed via devtools::install_github('CSkylarL/TimiGP'))

## Examples

```
devtools::install_github("CSkylarL/TimiGP"); library(TimiGP); info <- TimiCheckEvent(info); data <- TimiPrePropress(rna, markers, log = TRUE, GMNorm = TRUE)
```

## Evaluation signals

- TimiCheckEvent removes samples lacking valid event and time-to-event values; the filtered clinical table should contain only records with complete survival endpoints
- After TimiPrePropress with log = TRUE, expression values are log-transformed; verify values are on log scale
- After GMNorm = TRUE, each gene (marker) should have median-normalized (comparable) distribution across samples
- Output expression matrix rows should correspond exactly to markers in the input annotation (e.g., CellType_Charoentong2017_Bindea2013_Xu2018_Immune) and columns to cohort samples
- Downstream TimiGenePair and TimiCOX should run without errors on the preprocessed objects, yielding a marker pair score matrix

## Limitations

- TimiGP requires a large gene expression dataset with matched survival information to effectively identify prognostic gene pairs
- Power of the survival analysis is determined by the sample size, the number of events, and the quality of the survival information — poor clinical data quality degrades all downstream steps

## Evidence
