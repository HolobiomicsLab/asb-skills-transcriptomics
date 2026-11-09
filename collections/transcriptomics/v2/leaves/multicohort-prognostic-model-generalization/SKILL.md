---
name: multicohort-prognostic-model-generalization
description: Use when a prognostic model derived from gene-pair (IMGP) scores in a
  training cohort (e.g., TCGA SKCM06) must be validated for risk stratification across
  several independent bulk transcriptomic cohorts with matched survival data.
when_to_use_negative:
- Input cohorts lack matched event/time-to-event clinical information — TimiCheckEvent
  will remove low-quality samples and Cox fitting is impossible
- Data are single-cell expression matrices or cell-type abundance scores rather than
  bulk marker-gene expression from which pairwise marker scores can be computed
- The question is de novo discovery of cell-cell interactions rather than transferring
  an already-derived prognostic model to new cohorts
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: TimiGP
  role: R package providing TimiPrePropress, TimiCheckEvent, TimiGenePair and TimiCOX
    used to preprocess cohorts, build MPS matrices, and select prognostic marker pairs
  repo: https://github.com/CSkylarL/TimiGP
- name: R
  role: Execution environment for the TimiGP package and survival modeling
- name: survival
  role: Cox regression, Surv objects, Harrell's C-index and log-rank/Kaplan-Meier
    evaluation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.xcrm.2023.101121
    title: 'TimiGP: Inferring cell-cell interactions and prognostic associations in
      the tumor immune microenvironment through gene pairs'
  source_content_hash: sha256:3e6c48bdd2d1d3af493e2d2c57485fbfc3f0d08a3752279e31b8c80e65e72055
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/CSkylarL/TimiGP
  tools:
  - R
  - TimiGP
  - survival
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# multicohort-prognostic-model-generalization

## Summary

Fit a marker-pair-based Cox/LASSO prognostic risk score (TimiRS) in one training cohort and generalize it to multiple independent cohorts by applying fixed coefficients to each cohort's Marker Pair Score matrix, verifying Harrell's C-index, outperformance of a control model, and survival stratification.

## When to use

Use when a prognostic model derived from gene-pair (IMGP) scores in a training cohort (e.g., TCGA SKCM06) must be validated for risk stratification across several independent bulk transcriptomic cohorts with matched survival data.

## When NOT to use

- Input cohorts lack matched event/time-to-event clinical information — TimiCheckEvent will remove low-quality samples and Cox fitting is impossible
- Data are single-cell expression matrices or cell-type abundance scores rather than bulk marker-gene expression from which pairwise marker scores can be computed
- The question is de novo discovery of cell-cell interactions rather than transferring an already-derived prognostic model to new cohorts

## Inputs

- Training cohort clinical + transcriptomic data (TCGA SKCM06 metastatic melanoma: SKCM06info, SKCM06rna)
- Five independent validation cohorts (GSE65904, GSE22155, GSE54467, GSE53118, GSE8401) bulk RNA-seq/microarray expression
- Matched clinical survival info (event, time-to-event) per cohort
- CD8-Tem marker list / cell-type marker set (e.g., Immune_Marker_n1293)
- Pre-selected top 21 CD8-Tem-derived IMGPs and the 19 enriched cell-cell interaction annotations

## Outputs

- TimiRS LASSO Cox coefficients from the training cohort MPS matrix
- Per-patient TimiRS and TimiRS-control risk scores for each validation cohort
- Per-cohort Harrell's C-index values (expected 0.621-0.659)
- Median-split high/low-risk Kaplan-Meier/log-rank survival results per interaction
- timirs_validation_results.csv summarizing C-index, control comparison, and stratification

## How to apply

Preprocess each cohort with TimiPrePropress (log transformation and gene-wise median normalization over the cell-type marker list, e.g., CD8-Tem markers) and filter clinical data with TimiCheckEvent to drop low-quality event/time-to-event records. Build each cohort's Marker Pair Score (MPS) matrix via TimiGenePair, where each entry is the logical comparison 'gene A expression > gene B expression'. In the training cohort, use univariate Cox regression (TimiCOX) to select prognostic marker pairs, then fit a LASSO-penalized Cox regression (glmnet, alpha = 1) restricted to the top 21 CD8-Tem-derived IMGPs to obtain TimiRS coefficients. Apply these fixed coefficients to each validation cohort's MPS matrix (GSE65904, GSE22155, GSE54467, GSE53118, GSE8401) to compute per-patient risk scores, and in parallel compute a TimiRS-control score from marker pairs not associated with the 19 enriched cell-cell interactions. Evaluate with Harrell's C-index per cohort, median-split Kaplan-Meier/log-rank survival separation, and TimiRS vs. TimiRS-control comparison, assembling results into timirs_validation_results.csv.

## Related tools

- **TimiGP** (R package providing TimiPrePropress, TimiCheckEvent, TimiGenePair and TimiCOX used to preprocess cohorts, build MPS matrices, and select prognostic marker pairs) — https://github.com/CSkylarL/TimiGP
- **R** (Execution environment for the TimiGP package and survival modeling)
- **survival** (Cox regression, Surv objects, Harrell's C-index and log-rank/Kaplan-Meier evaluation)

## Examples

```
mps <- TimiGenePair(TimiPrePropress(rna, markers), markers); fit <- glmnet(mps_train, Surv(time, event), family = "cox", alpha = 1); risk <- as.vector(mps_val %*% coef(fit, s = "lambda.min"))
```

## Evaluation signals

- TimiRS C-index falls within the reported 0.621-0.659 range in each of the five validation cohorts (GSE65904, GSE22155, GSE54467, GSE53118, GSE8401)
- TimiRS C-index exceeds the TimiRS-control model C-index in every validation cohort
- The 19 identified cell-cell interactions stratify overall survival (KM/log-rank) in all five datasets
- MPS matrices are binary logical comparisons (1 = gene A expression > gene B expression) computed after log transformation and gene-wise median normalization
- Results remain consistent across independent melanoma datasets and across different cell-type marker set inputs

## Limitations

- TimiGP requires a large gene expression dataset with matched survival information to effectively identify prognostic gene pairs; power depends on sample size, number of events, and quality of survival information
- Interaction identification is enrichment-analysis based, so sensitivity depends on the number of marker genes; cell types with fewer marker genes are less likely to be identified
- No wet-lab experiments were performed to validate the findings, raising uncertainty about the reliability of the inferred cell-cell interactions

## Evidence

- [other] C-index target: [evidence span withheld — anchor status: not_found]
