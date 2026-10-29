---
name: lasso-cox-regression-feature-selection
description: Use when building an IMGP-based prognostic model and you need to test
  whether cell-cell-interaction-enriched gene pairs are preferentially retained by
  LASSO Cox regression compared with pairs chosen only for prognostic p-value significance.
when_to_use_negative:
- When the goal is direct cell-cell interaction inference or enrichment analysis itself
  rather than evaluating feature selection for a prognostic model
- When the cohort lacks matched survival information — TimiGP requires a large expression
  dataset with survival data to identify prognostic gene pairs
- When candidate gene pairs have not been precomputed from marker-gene pairwise comparisons;
  this skill operates on an existing IMGP candidate list
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0203
tools:
- name: TimiGP
  role: R package providing TimiGenePair, TimiCOX, TimiBG, TimiEnrich, TimiCheckEvent
    and TimiPrePropress used to generate candidate IMGP sets for LASSO comparison
  repo: https://github.com/CSkylarL/TimiGP
- name: R
  role: Environment in which the glmnet LASSO Cox cross-validation loop and selection-probability
    summarization are implemented
  repo: https://github.com/CSkylarL/MSofTimiGP
- name: survival
  role: R package underlying the Cox regression fits for prognostic IMGP selection
- name: glmnet
  role: Fits the LASSO Cox regression (alpha = 1) with 10-fold cross-validation to
    pick lambda in each of 1000 repetitions
- name: devtools
  role: Used to install the TimiGP package from GitHub
  repo: https://github.com/r-lib/devtools
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.xcrm.2023.101121
    title: 'TimiGP: Inferring cell-cell interactions and prognostic associations in
      the tumor immune microenvironment through gene pairs'
  source_content_hash: sha256:cb7819ccd5f1d0226ce532446aa151d9c7d4708803cf5a063a21325324b532d3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/CSkylarL/TimiGP
  tools:
  - R
  - TimiGP
  - devtools
  - glmnet
  - survival
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# lasso-cox-regression-feature-selection

## Summary

A repetition-based LASSO Cox regression feature-selection procedure that compares how often candidate immune marker gene pairs (IMGPs) are retained when drawn from TimiGP-enriched interaction-derived pairs versus pairs selected merely for univariate prognostic significance, quantifying whether TimiGP enrichment filters correlated but noisy features.

## When to use

Use when building an IMGP-based prognostic model and you need to test whether cell-cell-interaction-enriched gene pairs are preferentially retained by LASSO Cox regression compared with pairs chosen only for prognostic p-value significance.

## When NOT to use

- When the goal is direct cell-cell interaction inference or enrichment analysis itself rather than evaluating feature selection for a prognostic model
- When the cohort lacks matched survival information — TimiGP requires a large expression dataset with survival data to identify prognostic gene pairs
- When candidate gene pairs have not been precomputed from marker-gene pairwise comparisons; this skill operates on an existing IMGP candidate list

## Inputs

- TCGA SKCM06 clinical info (SKCM06info: event and time-to-event survival statistics)
- TCGA SKCM06 transcriptome profile (SKCM06rna)
- CellType_Charoentong2017_Bindea2013_Xu2018_Immune marker annotation
- 544 candidate IMGPs from prognostic (TimiCOX, BH-adjusted) and enriched (TimiBG/TimiEnrich, BH cutoff 0.05) pair sets

## Outputs

- Per-IMGP selection probability over 1000 LASSO repetitions, separately for prognostic-derived and enrichment-derived pair sets
- Fraction of the 544 candidate IMGPs more likely selected from enriched pairs (expected: 76%)
- Full probability table written to lasso_selection_probability.csv

## How to apply

Load the TCGA SKCM06 (metastatic melanoma) clinical info and transcriptome profile with the CellType_Charoentong2017_Bindea2013_Xu2018_Immune marker set, preprocess with TimiCheckEvent and TimiPrePropress (log = TRUE, GMNorm = TRUE), generate the Marker Pair Score matrix with TimiGenePair, then derive two candidate IMGP sets: prognostic pairs via TimiCOX univariate Cox regression (p.adj = 'BH', HR > 1 pairs reversed to B_A) and enriched pairs via TimiBG plus TimiEnrich (p.adj = 'BH', cutoff 0.05). For each of 1000 repetitions, fit a LASSO Cox model (glmnet, alpha = 1) with 10-fold cross-validation to choose the penalty lambda and record which of the 544 candidate IMGPs are retained. Compute per-IMGP selection probabilities for each derivation set and summarize the fraction of candidates whose selection probability is higher from enriched pairs.

## Related tools

- **TimiGP** (R package providing TimiGenePair, TimiCOX, TimiBG, TimiEnrich, TimiCheckEvent and TimiPrePropress used to generate candidate IMGP sets for LASSO comparison) — https://github.com/CSkylarL/TimiGP
- **R** (Environment in which the glmnet LASSO Cox cross-validation loop and selection-probability summarization are implemented) — https://github.com/CSkylarL/MSofTimiGP
- **survival** (R package underlying the Cox regression fits for prognostic IMGP selection)
- **glmnet** (Fits the LASSO Cox regression (alpha = 1) with 10-fold cross-validation to pick lambda in each of 1000 repetitions)
- **devtools** (Used to install the TimiGP package from GitHub) — https://github.com/r-lib/devtools

## Examples

```
devtools::install_github("CSkylarL/TimiGP"); library(TimiGP); mps <- TimiGenePair(gene, rna); cox.res <- TimiCOX(mps, info, p.adj="BH")
```

## Evaluation signals

- Approximately 76% of the 544 candidate IMGPs show a higher selection probability from enriched pairs than from prognostic pairs
- Each repetition's glmnet fit uses alpha = 1 and lambda chosen by 10-fold cross-validation; all 1000 repetitions must complete over the same 544-candidate IMGP space
- Prognostic pair derivation applies BH adjustment with HR > 1 pairs reversed to B_A before comparison; enriched pairs use BH-adjusted significance cutoff of 0.05
- lasso_selection_probability.csv contains the full per-IMGP probability table with one selection probability per candidate per derivation set

## Limitations

- TimiGP requires a large gene expression dataset with matched survival information to effectively identify prognostic gene pairs
- The power of the survival analysis is determined by the sample size, the number of events, and the quality of the survival information
- Results depend on the chosen cell-type marker set; sensitivity to identify a specific cell-cell interaction is affected by the number of corresponding marker genes since interactions are identified by enrichment analysis

## Evidence
