---
name: resampling-based-model-stability-analysis
description: Use when you need to demonstrate that biologically informed feature filtering
  (e.g., TimiGP-enriched cell-cell interaction pairs) makes LASSO Cox feature selection
  more stable than filtering on prognostic p-values alone.
when_to_use_negative:
- Input cohort lacks matched survival information or has too few events — TimiGP requires
  a large expression dataset with survival data to identify prognostic gene pairs
- Your goal is single-shot model fitting rather than assessing selection stability
  across resamples; a single glmnet fit suffices
- Feature candidates are already an independent, low-noise feature set where enrichment-based
  filtering adds no prior biological information
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0203
tools:
- name: TimiGP
  role: generates candidate marker gene pairs and derives both prognostic (TimiCOX)
    and enrichment-derived (TimiBG/TimiEnrich) feature sets being compared
  repo: https://github.com/CSkylarL/TimiGP
- name: R
  role: implementation language for the 1000-repetition LASSO Cox resampling loop
- name: survival
  role: underlying Cox regression machinery for univariate IMGP screening
- name: glmnet
  role: fits LASSO Cox regression (alpha=1) with 10-fold cross-validated lambda in
    each repetition
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.xcrm.2023.101121
    title: 'TimiGP: Inferring cell-cell interactions and prognostic associations in
      the tumor immune microenvironment through gene pairs'
  source_content_hash: sha256:85656588ddae63ad96cb9ee55724a4ef57f500a014b6bbc06d83494eb192e2e5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/CSkylarL/TimiGP
  tools:
  - R
  - TimiGP
  - glmnet
  - survival
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# resampling-based-model-stability-analysis

## Summary

Quantifies the stability of feature selection in an IMGP-based prognostic model by repeating LASSO Cox regression (glmnet, alpha=1) over 1000 resampling repetitions and comparing per-feature selection probabilities between TimiGP-enriched candidate pairs and pairs chosen only for univariate prognostic significance.

## When to use

Use when you need to demonstrate that biologically informed feature filtering (e.g., TimiGP-enriched cell-cell interaction pairs) makes LASSO Cox feature selection more stable than filtering on prognostic p-values alone.

## When NOT to use

- Input cohort lacks matched survival information or has too few events — TimiGP requires a large expression dataset with survival data to identify prognostic gene pairs
- Your goal is single-shot model fitting rather than assessing selection stability across resamples; a single glmnet fit suffices
- Feature candidates are already an independent, low-noise feature set where enrichment-based filtering adds no prior biological information

## Inputs

- Preprocessed transcriptome profile of TCGA SKCM06 (log-transformed, gene-wise median normalized via TimiPrePropress log=TRUE, GMNorm=TRUE)
- Clinical survival information filtered with TimiCheckEvent
- Logical marker pair score (MPS) matrix from TimiGenePair
- Prognostic IMGP set from TimiCOX (BH-adjusted p, HR>1 pairs reversed)
- Enriched IMGP set from TimiBG + TimiEnrich (p.adj='BH', cutoff 0.05)

## Outputs

- Per-IMGP selection probability table over 1000 repetitions for both feature sets (written to lasso_selection_probability.csv)
- Summary statistic: fraction of the 544 candidate IMGPs with higher selection probability from enriched pairs

## How to apply

Start from the TimiGenePair Marker Pair Score matrix (544 candidate IMGPs on TCGA SKCM06 with Charoentong2017 markers), derive two feature sets: prognostic pairs via TimiCOX (univariate Cox, BH-adjusted, HR>1 pairs reversed to B_A) and enriched pairs via TimiBG/TimiEnrich (BH, significance cutoff 0.05). In each of 1000 repetitions, fit a LASSO Cox regression with glmnet (alpha=1) using 10-fold cross-validation to select the penalty lambda, and record which candidate IMGPs are retained. Compute, per candidate IMGP, the fraction of repetitions in which it is selected, separately for each feature set. The rationale is that enrichment-derived pairs filter correlated but noisy features, so their selection probabilities should be systematically higher.

## Related tools

- **TimiGP** (generates candidate marker gene pairs and derives both prognostic (TimiCOX) and enrichment-derived (TimiBG/TimiEnrich) feature sets being compared) — https://github.com/CSkylarL/TimiGP
- **R** (implementation language for the 1000-repetition LASSO Cox resampling loop)
- **survival** (underlying Cox regression machinery for univariate IMGP screening)
- **glmnet** (fits LASSO Cox regression (alpha=1) with 10-fold cross-validated lambda in each repetition)

## Examples

```
library(glmnet); cvfit <- cv.glmnet(mps, Surv(time, event), family="cox", alpha=1, nfolds=10); sel <- which(as.numeric(coef(cvfit, s="lambda.min")) != 0)
```

## Evaluation signals

- At least 544 candidate IMGPs are scored, one selection probability per pair per feature set
- Repetitions are 1000, each with a 10-fold CV to pick lambda (glmnet alpha=1, family='cox')
- Validation criterion: approximately 76% of the 544 candidate IMGPs show higher selection probability from enriched pairs than from prognostic pairs
- Probability table is exported with a row per candidate IMGP (lasso_selection_probability.csv) enabling direct enriched-vs-prognostic comparison
- Directionality handling: pairs with HR > 1 are reversed to B_A in the prognostic screen so both sets are comparable

## Limitations

- TimiGP requires a large gene expression dataset with matched survival information to effectively identify prognostic gene pairs; power depends on sample size, number of events, and survival data quality
- Results are cohort- and marker-set-dependent (TCGA_SKCM06, CellType_Charoentong2017_Bindea2013_Xu2018_Immune); the 76% figure may not transfer to other cohorts
- Resampling-based stability estimates are stochastic without a fixed seed, so exact per-pair probabilities may vary across runs

## Evidence

- [other] workflow glmnet parameters: [evidence span withheld — anchor status: not_found]
- [other] validation criterion: [evidence span withheld — anchor status: not_found]
- [other] rationale for stability gain: [evidence span withheld — anchor status: not_found]
