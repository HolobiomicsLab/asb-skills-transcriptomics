---
name: cell-cell-interaction-risk-score-modeling
description: Use when bulk transcriptomes with matched survival data (e.g., TCGA SKCM06
  plus GSE validation cohorts) are available and a prognostic model must aggregate
  interaction-linked marker gene pairs into one risk score.
when_to_use_negative:
- The cohort lacks matched survival information — TimiGP needs a large expression
  dataset with survival data to identify prognostic gene pairs
- You need direct cell-type abundance estimation from single-cell-resolution data
  rather than bulk-inferred interaction/prognosis associations
- You only need to infer/annotate the interaction network itself (TimiGeneNetwork/TimiEnrich),
  not turn it into a predictive risk score
edam_topics:
- http://edamontology.org/topic_0634
tools:
- name: TimiGP
  role: R package providing TimiPrePropress, TimiCheckEvent, TimiGenePair (MPS matrix)
    and TimiCOX (univariate Cox) for the pair-based prognostic pipeline
  repo: https://github.com/CSkylarL/TimiGP
- name: R
  role: Implementation language for the TimiGP package, glmnet LASSO Cox fitting,
    and survival package Kaplan-Meier/log-rank evaluation
- name: survival
  role: 'Survival analysis: Harrell''s C-index computation and Kaplan-Meier/log-rank
    assessment of risk-group stratification'
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.xcrm.2023.101121
    title: 'TimiGP: Inferring cell-cell interactions and prognostic associations in
      the tumor immune microenvironment through gene pairs'
  source_content_hash: sha256:493a5abf07124ff0077e95bc1b816e8516fa729bcb2d320b8ad45bf106b41077
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/CSkylarL/TimiGP
  tools:
  - R
  - TimiGP
  - survival
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0634
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# cell-cell-interaction-risk-score-modeling

## Summary

Builds an interpretable survival risk score (TimiRS-style) from LASSO-penalized Cox regression restricted to cell-cell-interaction-associated marker gene pairs (IMGPs), rather than single interactions, and validates stratification across independent bulk-tumor cohorts.

## When to use

Use when bulk transcriptomes with matched survival data (e.g., TCGA SKCM06 plus GSE validation cohorts) are available and a prognostic model must aggregate interaction-linked marker gene pairs into one risk score.

## When NOT to use

- The cohort lacks matched survival information — TimiGP needs a large expression dataset with survival data to identify prognostic gene pairs
- You need direct cell-type abundance estimation from single-cell-resolution data rather than bulk-inferred interaction/prognosis associations
- You only need to infer/annotate the interaction network itself (TimiGeneNetwork/TimiEnrich), not turn it into a predictive risk score

## Inputs

- Bulk transcriptome profile (RNA-seq or microarray) for a training cohort and ≥1 independent validation cohorts (e.g., TCGA SKCM06; GSE65904, GSE22155, GSE54467, GSE53118, GSE8401)
- Clinical information with event and time-to-event fields (e.g., SKCM06info)
- Cell-type marker list (e.g., CD8-Tem markers, or Immune_Marker_n1293 / scRNA-seq-derived markers)
- Set of candidate interaction-associated marker gene pairs (top 21 CD8-Tem-derived IMGPs; 19 enriched cell-cell interactions)

## Outputs

- Per-patient TimiRS risk score per cohort (coefficient-weighted sum of binary marker pair scores)
- LASSO Cox coefficients for the selected marker gene pairs
- Harrell's C-index per validation cohort, plus TimiRS-control comparison
- High- vs low-risk stratification with Kaplan-Meier/log-rank results per cohort
- Aggregated results table (e.g., timirs_validation_results.csv)

## How to apply

Preprocess each cohort with TimiPrePropress (log transformation and gene-wise median normalization restricted to the marker list, e.g., CD8-Tem markers) and filter clinical data with TimiCheckEvent to remove low-quality event/time-to-event records. Convert expression to a binary Marker Pair Score (MPS) matrix with TimiGenePair, where 1 means gene A expression > gene B for each marker pair. In the training cohort (TCGA SKCM06), run univariate Cox regression (TimiCOX) on each marker pair, then fit a LASSO-penalized Cox model (glmnet, alpha = 1, family cox) restricted to the interaction-linked pairs (e.g., top 21 CD8-Tem-derived IMGPs) to obtain risk-score coefficients. Apply these coefficients to each validation cohort's MPS matrix to compute per-patient TimiRS scores, and build a control model from marker pairs not associated with the identified cell-cell interactions (e.g., the 19 enriched interactions) for comparison. Judge performance by Harrell's C-index per cohort, median risk-score split with Kaplan-Meier/log-rank separation, and superiority over the control model.

## Related tools

- **TimiGP** (R package providing TimiPrePropress, TimiCheckEvent, TimiGenePair (MPS matrix) and TimiCOX (univariate Cox) for the pair-based prognostic pipeline) — https://github.com/CSkylarL/TimiGP
- **R** (Implementation language for the TimiGP package, glmnet LASSO Cox fitting, and survival package Kaplan-Meier/log-rank evaluation)
- **survival** (Survival analysis: Harrell's C-index computation and Kaplan-Meier/log-rank assessment of risk-group stratification)

## Examples

```
fit <- glmnet(x = t(mps_SKCM06[, top21_IMGPs]), y = Surv(time, event), family = "cox", alpha = 1); risk <- t(mps_valid) %*% coef(fit, s = "lambda.min")
```

## Evaluation signals

- TimiRS C-index must fall within the reported 0.621-0.659 range across the five independent validation cohorts (GSE65904, GSE22155, GSE54467, GSE53118, GSE8401)
- TimiRS C-index must exceed the TimiRS-control model C-index in validation cohorts
- The 19 identified cell-cell interactions must stratify overall survival (median-split high/low risk) in all five datasets
- MPS matrix must be binary per TimiGenePair's logical rule (1 = gene A expression > gene B expression)
- LASSO must be fit with alpha = 1 (glmnet) and restricted to the top 21 CD8-Tem-derived IMGPs, not all marker pairs

## Limitations

- To effectively identify prognostic gene pairs, TimiGP requires a large gene expression dataset with matched survival information
- Power of the survival analysis is determined by sample size, number of events, and quality of the survival information
- Because interactions are identified by enrichment analysis, sensitivity to detect a specific cell-cell interaction depends on the number of its marker genes; cell types with fewer markers are less likely to be detected
- No wet-lab experiments were performed to validate the important findings, raising uncertainty about the reliability of the interaction inference

## Evidence
