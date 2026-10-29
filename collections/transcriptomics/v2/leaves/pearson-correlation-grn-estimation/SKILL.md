---
name: pearson-correlation-grn-estimation
description: Use when benchmarking a cell-specific network inference method against
  dyngen's per-cell ground-truth regulatory networks and you need a simple correlation-differencing
  baseline (SSN) using Pearson correlation as the base estimator.
when_to_use_negative:
- The task requires method discovery or high benchmark performance rather than a simple
  negative-control baseline — SSN performed poorly because its subtract-one-cell correlation
  differences are mostly noise.
- The data lack per-cell ground-truth regulatory networks, so AUROC/AUPR against a
  known CSN cannot be computed.
- The analysis goal is RNA velocity or trajectory alignment benchmarking, the other
  dyngen applications.
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: dyngen
  role: Generates the 42 simulated benchmark datasets with cell-specific ground-truth
    regulatory networks used to score the Pearson/SSN predictions
  repo: https://github.com/dynverse/dyngen
- name: tidyverse
  role: R data-manipulation toolkit used to implement the SSN leave-one-out differencing
    and aggregate per-cell scores
- name: R
  role: Implementation language for the SSN baseline and AUROC/AUPR evaluation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-24152-2
    title: Spearheading future omics analyses using dyngen, a multi-modal simulator
      of single cells
  source_content_hash: sha256:045f433205d279460aee8ca11aa70512b7faea59a1d0e683fb28440f5178e70b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dynverse/dyngen
  tools:
  - R
  - dyngen
  - tidyverse
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# pearson-correlation-grn-estimation

## Summary

A cell-specific gene regulatory network (CSNI) baseline that scores every TF–gene pair per cell using a leave-one-out differencing of Pearson correlation matrices (the SSN approach), evaluated against dyngen's cell-specific ground-truth GRNs on the 42-dataset benchmark.

## When to use

Use when benchmarking a cell-specific network inference method against dyngen's per-cell ground-truth regulatory networks and you need a simple correlation-differencing baseline (SSN) using Pearson correlation as the base estimator.

## When NOT to use

- The task requires method discovery or high benchmark performance rather than a simple negative-control baseline — SSN performed poorly because its subtract-one-cell correlation differences are mostly noise.
- The data lack per-cell ground-truth regulatory networks, so AUROC/AUPR against a known CSN cannot be computed.
- The analysis goal is RNA velocity or trajectory alignment benchmarking, the other dyngen applications.

## Inputs

- 42 dyngen simulated datasets (14 backbones x 3 seeds) with per-cell transcriptomics count/expression matrices
- Per-cell ground-truth cell-specific regulatory networks (TF–gene interaction lists) from dyngen

## Outputs

- Per-cell predicted CSNI networks (top 10,000 scored TF–gene interactions per cell)
- Per-dataset mean AUROC and mean AUPR scores for the SSN condition

## How to apply

Load each of the 42 dyngen benchmark datasets (14 predefined backbones across three seeds), which provide per-cell transcriptomics profiles and the corresponding cell-specific ground-truth regulatory networks. For each dataset, implement SSN in R with basic R and tidyverse functions following the same leave-one-out differencing approach as LIONESS, but restricted to the Pearson correlation as the base network inference method: for every cell, compute correlation matrices on the full dataset and on the dataset minus that cell, and use the difference to score interactions. For every cell, retain the top 10,000 predicted interactions by score. Compare each cell's predicted network against the ground-truth cell-specific GRN and compute AUROC and AUPR, then average the per-cell scores within each dataset to obtain mean AUROC and mean AUPR for the SSN condition. The rationale for this baseline is that the paper found SSN's subtract-one-cell correlation differences are mostly noise, so it performs worse than LIONESS and pySCENIC on the dyngen CSNI benchmark.

## Related tools

- **dyngen** (Generates the 42 simulated benchmark datasets with cell-specific ground-truth regulatory networks used to score the Pearson/SSN predictions) — https://github.com/dynverse/dyngen
- **tidyverse** (R data-manipulation toolkit used to implement the SSN leave-one-out differencing and aggregate per-cell scores)
- **R** (Implementation language for the SSN baseline and AUROC/AUPR evaluation)

## Examples

```
Rscript -e 'full <- cor(t(expr), method="pearson"); loo <- cor(t(expr[-i,]), method="pearson"); ssn_scores <- full - loo; top10000 <- head(order(-ssn_scores), 10000)'
```

## Evaluation signals

- Each of the 42 datasets yields a mean AUROC and mean AUPR value in [0, 1].
- The resulting scores table contains an entry for every dataset in the SSN condition (42 rows).
- Every cell's predicted network contains exactly the top 10,000 interactions by score.
- Scores are computed over the retained top 10,000 interactions per cell against the dyngen cell-specific ground-truth GRN.
- SSN's mean AUROC/AUPR should fall below LIONESS and pySCENIC on the same dyngen benchmark, consistent with the reported negative result.

## Limitations

- SSN performs poorly on the dyngen CSNI benchmark: subtracting one cell from the whole dataset yields correlation differences that are mostly noise.
- The leave-one-out differencing is restricted to Pearson correlation, so it captures only linear co-expression signal, not directed or dynamic regulation.
- dyngen assumes cells are well-mixed and independent from each other, and no software version numbers are reported for the tools used.

## Evidence

- [other] SSN performs poorly on the dyngen CSNI benchmark because its approach of subtracting one cell from the whole dataset yields correlation differences that are mostly noise, performing worse than LIONESS and pySCENIC.: [evidence span withheld — anchor status: not_found]
- [results] Per-cell top-k and AUROC/AUPR evaluation protocol: "For each dataset, we computed the mean Area Under the Receiver Operating Characteristic-curve (AUROC) and Area Under the Precision-Recall curve (AUPR) scores of the individual cells"
- [readme] Manuscript reproduction scripts location: [evidence span withheld — anchor status: not_found]
- [abstract] dyngen ground-truth CSNI purpose: "three applications: aligning cell developmental trajectories, cell-specific regulatory network inference and estimation of RNA velocity"
