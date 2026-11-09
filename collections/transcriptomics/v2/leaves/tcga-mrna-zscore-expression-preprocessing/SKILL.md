---
name: tcga-mrna-zscore-expression-preprocessing
description: Use when raw TCGA mRNA expression data must be converted to Z-scores
  and filtered (abnormal if Z > 2 or Z < -2; entries abnormal in >= 2% of patients
  retained) before survival or binary-outcome regression.
when_to_use_negative:
- Input data are already Z-scored and frequency-filtered, e.g., a processed expression
  table ready for penalized Cox regression
- The variable set is small enough that the multiple-testing problem does not apply
  and penalized regression can be fit directly without preliminary screening
- The analysis targets non-TCGA or non-genomic data types incompatible with the mRNA
  expression Z-score paradigm
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_2640
tools:
- name: HDMAC
  role: Web-based platform implementing the preliminary screening of mRNA entries
    by abnormal-expression frequency prior to penalized regression
  repo: https://github.com/chung-R/HD-MAC
- name: R
  role: Language used for the platform, statistical analysis, and the custom scripts
    to download large numbers of TCGA genomic data
- name: RStudio
  role: Offline execution of HDMAC R scripts for data download and preprocessing
- name: cBioPortal
  role: Alternative source of TCGA data, limited by per-request download size
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-020-60791-z
    title: 'HDMAC: A Web-Based Interactive Program for High-Dimensional Analysis of
      Molecular Alterations in Cancer'
  source_content_hash: sha256:5b72ddf7d54a36174d6d4d53e059dad7bb8ea3c93f491c52efb68af48585528c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chung-R/HD-MAC
  tools:
  - HDMAC
  - R
  - RStudio
  - cBioPortal
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# tcga-mrna-zscore-expression-preprocessing

## Summary

Preprocessing of TCGA mRNA expression data using Z-score thresholds to flag abnormal expression entries and retain informative genes for downstream penalized regression, as implemented in the HDMAC workflow.

## When to use

Use when raw TCGA mRNA expression data must be converted to Z-scores and filtered (abnormal if Z > 2 or Z < -2; entries abnormal in >= 2% of patients retained) before survival or binary-outcome regression.

## When NOT to use

- Input data are already Z-scored and frequency-filtered, e.g., a processed expression table ready for penalized Cox regression
- The variable set is small enough that the multiple-testing problem does not apply and penalized regression can be fit directly without preliminary screening
- The analysis targets non-TCGA or non-genomic data types incompatible with the mRNA expression Z-score paradigm

## Inputs

- Raw TCGA mRNA expression matrix (patients x gene entries) downloaded via the HDMAC R scripts
- Clinical covariates such as overall survival time and censoring indicators (for survival endpoints)
- Patient mutation status matrix (for the parallel 1% mutation-frequency screen)

## Outputs

- Z-scored mRNA expression matrix filtered to entries abnormal in >= 2% of patients (e.g., 316 patients x 9,548 entries for TCGA ovarian HGSC)
- Vector of abnormality flags per entry per patient (Z > 2 or Z < -2)

## How to apply

Download the TCGA genomic data using the custom R scripts provided by the HDMAC project (github.com/chung-R/HD-MAC), which support bulk download of large numbers of molecular variables that cBioPortal limits per request. Z-score each mRNA expression entry so that a Z score above 2 or below -2 is deemed abnormal. Apply a preliminary frequency screen retaining only mRNA entries whose abnormal expression is observed in 2% or more of the patients (the analogous mutation screen uses 1%). Output the retained Z-scored expression matrix (e.g., 316 TCGA ovarian HGSC patients x 9,548 entries) for pairing with overall survival times and censoring indicators or binary outcomes for Ridge, Lasso, and adaptive Lasso penalized regression.

## Related tools

- **HDMAC** (Web-based platform implementing the preliminary screening of mRNA entries by abnormal-expression frequency prior to penalized regression) — https://github.com/chung-R/HD-MAC
- **R** (Language used for the platform, statistical analysis, and the custom scripts to download large numbers of TCGA genomic data)
- **RStudio** (Offline execution of HDMAC R scripts for data download and preprocessing)
- **cBioPortal** (Alternative source of TCGA data, limited by per-request download size)

## Examples

```
Rscript download_tcga_data.R  # then screen: z <- scale(expr); abnormal <- z > 2 | z < -2; keep <- colMeans(abnormal) >= 0.02
```

## Evaluation signals

- Every retained mRNA entry is abnormal (Z > 2 or Z < -2) in at least 2% of the patients
- Processed matrix dimensions match the reference: 316 TCGA ovarian HGSC patients and 9,548 retained Z-scored mRNA expression entries
- All values in the output matrix are Z-scores comparable across entries (mean-centered, variance-normalized per gene)
- Retained entries align one-to-one with patients' survival time and censoring indicator vectors downstream

## Limitations

- The 2% (expression) and 1% (mutation) frequency cutoffs are heuristic preliminary screens; genes passing them are only candidates requiring further in-depth study
- Users dealing with variables at the magnitude of a hundred thousand should include FDR screening, since penalized regression models fall short at that scale
- The preliminary list of candidate genes is platform- and cohort-specific and must be re-derived for new datasets

## Evidence

- [methods] A Z score above 2 or below −2 was considered abnormal: "A Z score above 2 or below −2 was considered abnormal"
- [methods] Frequency screen of mRNA entries: "a preliminary screening was performed so that only the genes whose mutations were found in 1% or more and the mRNA entries whose abnormal expression was found in 2% or more of the patients were"
- [methods] Custom R download scripts: "We therefore wrote R codes to download large numbers of genomic data from TCGA"
- [abstract] Optional first-step screening rationale: "Choice of a first-step screening is provided to address the multiple-comparison issue that often arises with large-volume genomic data."
- [discussion] FDR screening for very large variable sets: "Inclusion of FDR is recommended if users are dealing with variables at the magnitude of a hundred thousand where penalized regression models fall short"
