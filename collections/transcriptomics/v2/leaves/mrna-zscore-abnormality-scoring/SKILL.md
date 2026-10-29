---
name: mrna-zscore-abnormality-scoring
description: Use when raw TCGA mRNA expression matrices must be reduced to abnormal-expression
  entries before FDR screening or Ridge/Lasso/adaptive-Lasso penalized regression
  in HDMAC.
when_to_use_negative:
- Data are mutation calls rather than mRNA expression — the article applies a 1%-mutation
  frequency screen for mutations, not the 2% mRNA abnormal-expression Z-score rule
- The covariate count is modest and penalized regression or abnormality pre-screening
  is unnecessary
- Clinical variables (gender, age) alone are the analysis target, not high-dimensional
  molecular entries
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: R
  role: Statistical computing environment used to download TCGA mRNA data and perform
    the Z-score standardization and 2% abnormality frequency screening
  repo: https://github.com/chung-R/HD-MAC
- name: HDMAC
  role: Web-based platform whose penalized-regression pipeline consumes the pre-screened
    abnormal-expression mRNA entries produced by this step
  repo: https://github.com/chung-R/HD-MAC
provenance:
  source_task_ids:
  - task_003
  - task_002
  - task_001
  - task_005
  source_papers:
  - doi: 10.1038/s41598-020-60791-z
    title: 'HDMAC: A Web-Based Interactive Program for High-Dimensional Analysis of
      Molecular Alterations in Cancer'
  source_content_hash: sha256:02bee38b029519ad11934868e0035aae18800047397724262cf9130a71f52537
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chung-R/HD-MAC
  tools:
  - HDMAC
  - R
  - Shiny
  - selectiveInference
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# mrna-zscore-abnormality-scoring

## Summary

Pre-screening step from the HDMAC workflow that converts TCGA mRNA expression values into Z-scores, flags entries as abnormally expressed when Z > 2 or Z < -2, and retains only entries abnormal in at least 2% of patients (8,024 entries for bladder, 9,548 for ovarian cancer) before penalized regression.

## When to use

Use when raw TCGA mRNA expression matrices must be reduced to abnormal-expression entries before FDR screening or Ridge/Lasso/adaptive-Lasso penalized regression in HDMAC.

## When NOT to use

- Data are mutation calls rather than mRNA expression — the article applies a 1%-mutation frequency screen for mutations, not the 2% mRNA abnormal-expression Z-score rule
- The covariate count is modest and penalized regression or abnormality pre-screening is unnecessary
- Clinical variables (gender, age) alone are the analysis target, not high-dimensional molecular entries

## Inputs

- TCGA mRNA expression matrix (patients x mRNA entries) downloaded via the HDMAC R scripts
- Patient cohort size (e.g., 189 bladder cancer patients, 316 ovarian cancer patients)

## Outputs

- Z-scored mRNA expression matrix with binary abnormal-expression indicators (Z > 2 or Z < -2)
- Pre-screened expression matrix restricted to entries abnormal in >= 2% of patients (8,024 entries bladder; 9,548 entries ovarian)

## How to apply

Download the mRNA expression matrix from TCGA (e.g., 189 bladder cancer or 316 ovarian cancer patients) using the HDMAC-provided R codes, then standardize each mRNA entry's expression values into Z-scores. Classify a patient's expression of an entry as abnormal when the Z score is above 2 or below -2. Apply the frequency filter: keep only mRNA entries whose abnormal expression is observed in 2% or more of the patients, which reduces the data to 8,024 entries (bladder) or 9,548 entries (ovarian). This preliminary screening shrinks the covariate space before univariate FDR first-step screening and penalized logistic or Cox regression, and the retained binary abnormal/normal indicator matrix (or coefficient-weighted Z-scores) feeds downstream HDMAC model fitting and cross-validation.

## Related tools

- **R** (Statistical computing environment used to download TCGA mRNA data and perform the Z-score standardization and 2% abnormality frequency screening) — https://github.com/chung-R/HD-MAC
- **HDMAC** (Web-based platform whose penalized-regression pipeline consumes the pre-screened abnormal-expression mRNA entries produced by this step) — https://github.com/chung-R/HD-MAC

## Examples

```
Rscript download_tcga_mrna.R --cancer bladder; z <- scale(expr); abnormal <- (z > 2) | (z < -2); keep <- colMeans(abnormal) >= 0.02
```

## Evaluation signals

- For the TCGA bladder cohort (189 patients), the screened matrix contains exactly 8,024 retained mRNA entries; for the ovarian cohort (316 patients), 9,548 entries
- Every retained entry shows abnormal expression (|Z| > 2) in at least 2% of patients; entries failing this frequency must be dropped
- Z-score transform must be recomputed per mRNA entry across patients, with abnormal called only at Z > 2 or Z < -2 (strict, not >= / <=)
- Downstream FDR first-step screening and penalized regression (Ridge/Lasso/adaptive Lasso) should run on the pre-screened entries and reproduce reported cross-validated AUCs (e.g., adaptive Lasso with FDR: 64.300)

## Limitations

- Penalized regression models fall short at variables of the magnitude of a hundred thousand, so FDR first-step screening is recommended in addition to this frequency screen
- The 2% abnormality and |Z| > 2 cutoffs are the article's defaults; cohorts smaller than ~50 patients make the 2% threshold retain very few entries
- Genes identified after this screening are explicitly preliminary and require further in-depth study

## Evidence

- [methods] A Z score above 2 or below −2 was considered abnormal: "A Z score above 2 or below −2 was considered abnormal"
- [methods] Preliminary frequency screening: "a preliminary screening was performed so that only the genes whose mutations were found in 1% or more and the mRNA entries whose abnormal expression was found in 2% or more of the patients were"
- [methods] TCGA download R codes: "We therefore wrote R codes to download large numbers of genomic data from TCGA"
- [methods] FDR as optional first-step screening: "the method to control the FDR is provided as an optional first-step screening method"
- [discussion] Penalized regression limits at ~100k variables: "Inclusion of FDR is recommended if users are dealing with variables at the magnitude of a hundred thousand where penalized regression models fall short"
