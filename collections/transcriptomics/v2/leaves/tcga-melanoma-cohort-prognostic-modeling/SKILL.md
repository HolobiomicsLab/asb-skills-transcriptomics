---
name: tcga-melanoma-cohort-prognostic-modeling
description: Use when building an IMGP-based prognostic model from bulk TCGA melanoma
  (SKCM06) expression and survival data, and you want to test whether filtering candidate
  gene pairs by cell-cell interaction enrichment improves LASSO Cox feature selection.
when_to_use_negative:
- The cohort is small or lacks matched survival information — TimiGP requires a large
  gene expression dataset with matched survival information to effectively identify
  prognostic gene pairs.
- The outcome of interest is therapy response rather than prognosis — the Response
  Module (TimiGP-Response) is the appropriate module.
- The input is already a deconvolved cell-type abundance table or single-cell data
  rather than bulk transcriptomic profiles with cell-type marker pairs.
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: TimiGP
  role: R package providing TimiCheckEvent, TimiPrePropress, TimiGenePair, TimiCOX,
    TimiBG and TimiEnrich used to preprocess the SKCM06 cohort, generate the marker
    pair score matrix, derive prognostic and interaction-enriched IMGP sets
  repo: https://github.com/CSkylarL/TimiGP
- name: MSofTimiGP
  role: Repository of R codes and intermediate data (.rda, .csv) reproducing the manuscript
    analyses, including the LASSO selection-probability workflow
  repo: https://github.com/CSkylarL/MSofTimiGP
- name: glmnet
  role: Fits the LASSO Cox regression (alpha = 1) with 10-fold cross-validation to
    select penalty lambda in each of the 1000 repetitions
- name: survival
  role: Supports the univariate Cox regression fits on each marker pair within TimiCOX
- name: devtools
  role: Used to install the TimiGP package from GitHub (devtools::install_github)
  repo: https://github.com/r-lib/devtools
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.xcrm.2023.101121
    title: 'TimiGP: Inferring cell-cell interactions and prognostic associations in
      the tumor immune microenvironment through gene pairs'
  source_content_hash: sha256:c8cb2ce82a6e39ea386cf6b2211df700314a0c1614123f453991d8a92279eba0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/CSkylarL/TimiGP
  tools:
  - MSofTimiGP
  - R
  - TimiGP
  - devtools
  - glmnet
  - survival
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# tcga-melanoma-cohort-prognostic-modeling

## Summary

Builds an IMGP-based prognostic model on the TCGA SKCM06 metastatic melanoma cohort and demonstrates that LASSO Cox feature selection is more likely to retain immune marker gene pairs (IMGPs) derived from TimiGP-enriched cell-cell interactions (76% of 544 candidates) than pairs chosen solely for univariate prognostic significance.

## When to use

Use when building an IMGP-based prognostic model from bulk TCGA melanoma (SKCM06) expression and survival data, and you want to test whether filtering candidate gene pairs by cell-cell interaction enrichment improves LASSO Cox feature selection.

## When NOT to use

- The cohort is small or lacks matched survival information — TimiGP requires a large gene expression dataset with matched survival information to effectively identify prognostic gene pairs.
- The outcome of interest is therapy response rather than prognosis — the Response Module (TimiGP-Response) is the appropriate module.
- The input is already a deconvolved cell-type abundance table or single-cell data rather than bulk transcriptomic profiles with cell-type marker pairs.

## Inputs

- TCGA SKCM06 (metastatic melanoma) clinical info (SKCM06info: event and time-to-event)
- TCGA SKCM06 transcriptome profile (SKCM06rna: bulk RNA-seq expression matrix)
- CellType_Charoentong2017_Bindea2013_Xu2018_Immune marker annotation (immune cell types and markers from 3 publications)

## Outputs

- Per-IMGP selection probabilities over 1000 LASSO repetitions for prognostic-derived and enrichment-derived pair sets
- Fraction of the 544 candidate IMGPs whose selection probability is higher from enriched pairs (expected criterion: ~76%)
- Full probability table written to lasso_selection_probability.csv

## How to apply

Load the TCGA_SKCM06 clinical info (SKCM06info) and transcriptome profile (SKCM06rna) together with the CellType_Charoentong2017_Bindea2013_Xu2018_Immune marker annotation, and preprocess with TimiCheckEvent (filtering low-quality event/time-to-event data) and TimiPrePropress with log = TRUE and GMNorm = TRUE (log transformation plus gene-wise median normalization). Generate the logical Marker Pair Score (MPS) matrix with TimiGenePair, then derive two candidate pair sets: prognostic IMGPs via TimiCOX (univariate Cox regression per marker pair, BH p.adj, pairs with HR > 1 reversed to B_A) and enriched IMGPs via TimiBG (background gene pairs) and TimiEnrich (BH adjustment, significance cutoff 0.05). In each of 1000 repetitions, fit a LASSO Cox regression (glmnet with alpha = 1) using 10-fold cross-validation to pick the penalty lambda and record which of the 544 candidate IMGPs are retained. Per candidate IMGP, compute selection probability separately for the prognostic-derived and enrichment-derived pair sets, then summarize the fraction of candidates whose selection probability is higher from enriched pairs; the reported criterion is 76% of the 544 IMGPs. The rationale is that interaction-enriched pairs filter correlated but noisy features that univariate significance alone does not remove.

## Related tools

- **TimiGP** (R package providing TimiCheckEvent, TimiPrePropress, TimiGenePair, TimiCOX, TimiBG and TimiEnrich used to preprocess the SKCM06 cohort, generate the marker pair score matrix, derive prognostic and interaction-enriched IMGP sets) — https://github.com/CSkylarL/TimiGP
- **MSofTimiGP** (Repository of R codes and intermediate data (.rda, .csv) reproducing the manuscript analyses, including the LASSO selection-probability workflow) — https://github.com/CSkylarL/MSofTimiGP
- **glmnet** (Fits the LASSO Cox regression (alpha = 1) with 10-fold cross-validation to select penalty lambda in each of the 1000 repetitions)
- **survival** (Supports the univariate Cox regression fits on each marker pair within TimiCOX)
- **devtools** (Used to install the TimiGP package from GitHub (devtools::install_github)) — https://github.com/r-lib/devtools

## Examples

```
TimiPrePropress(info = SKCM06info, rna = SKCM06rna, marker = CellType_Charoentong2017_Bindea2013_Xu2018_Immune, log = TRUE, GMNorm = TRUE)
```

## Evaluation signals

- 76% of the 544 candidate IMGPs show a higher LASSO selection probability from enriched pairs than from prognostic pairs (the reported validation criterion).
- The full probability table lasso_selection_probability.csv contains one row per candidate IMGP with both prognostic-derived and enrichment-derived selection probabilities over 1000 repetitions.
- Every gene pair in the prognostic set passes BH-adjusted univariate Cox regression, and pairs with HR > 1 are reversed to the B_A orientation before downstream use.
- Enriched IMGPs are those passing TimiEnrich with BH-adjusted significance cutoff 0.05 against TimiBG-generated background gene pairs.
- Each LASSO fit uses alpha = 1 with a 10-fold cross-validated lambda over exactly 1000 repetitions, and only the 544 candidate IMGPs are eligible for retention.

## Limitations

- TimiGP requires a large gene expression dataset with matched survival information to effectively identify prognostic gene pairs.
- No wet-lab experiments were performed to validate the important findings, raising uncertainty about the reliability of the inferred cell-cell interactions.
- Because cell-cell interactions are identified by enrichment analysis, sensitivity to detect a specific interaction depends on the number of its marker genes; cell types with fewer markers are less likely to yield significant interactions.

## Evidence
