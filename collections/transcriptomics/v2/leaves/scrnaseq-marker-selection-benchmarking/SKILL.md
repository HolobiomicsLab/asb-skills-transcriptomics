---
name: scrnaseq-marker-selection-benchmarking
description: 'Use when comparing marker-selection methods on scRNA-seq phenotype-classification
  tasks: you have count matrices with cell-type/phenotype labels and need discriminative
  marker sets plus downstream classifier F1 scores.'
when_to_use_negative:
- Input is already a curated marker list or feature table — selection/benchmarking
  is unnecessary
- The task is deconvolution or pseudo-bulk mixture reconstruction rather than marker-set
  classification benchmarking
- Only a single phenotype pair exists and a simple per-pair DE ranking suffices; the
  PC formulation targets many phenotype pairs with overlapping DE genes
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2269
tools:
- name: phenotype-cover
  role: Provides GreedyPC and CEMPC for phenotype-cover marker selection; installed
    via pip install phenotype-cover
  repo: https://github.com/euxhenh/phenotype-cover
- name: multiset-multicover
  role: Dependency package for phenotype-cover; PC is shown to be equivalent to multiset
    multicover
- name: scikit-learn
  role: Logistic regression classifier for evaluation, plus decision-tree and mutual-information
    baseline selectors
- name: mrmr_selection
  role: mRMR baseline; use mrmr_classif(X=X, y=y, K=k) to select k categorical features
    on the same splits
  repo: https://github.com/smazzanti/mrmr
- name: numpy
  role: Preprocessing of count matrices (gene/cell filtering, normalization, log transform)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100332
    title: Multiset multicover methods for discriminative marker selection
  source_content_hash: sha256:12c2c9f2b548d7c94d9efc785c764efd47b2ce75bdaed95ae33110691d20cac1
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/euxhenh/phenotype-cover
  repo_url: https://github.com/euxhenh/phenotype-cover
  tools:
  - GreedyPC
  - Python
  - mrmr_selection
  - multiset-multicover
  - numpy
  - phenotype-cover
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# scrnaseq-marker-selection-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark phenotype-cover (G-PC/CEM-PC) marker selection against baselines (scGeneFit, decision trees, mutual information, TopDE, ReliefF, mRMR) on scRNA-seq datasets by training logistic regression on selected markers and scoring macro-F1 on left-out cells.

## When to use

Use when comparing marker-selection methods on scRNA-seq phenotype-classification tasks: you have count matrices with cell-type/phenotype labels and need discriminative marker sets plus downstream classifier F1 scores.

## When NOT to use

- Input is already a curated marker list or feature table — selection/benchmarking is unnecessary
- The task is deconvolution or pseudo-bulk mixture reconstruction rather than marker-set classification benchmarking
- Only a single phenotype pair exists and a simple per-pair DE ranking suffices; the PC formulation targets many phenotype pairs with overlapping DE genes

## Inputs

- Preprocessed scRNA-seq count matrices with phenotype/cell-type labels (IPF Adams et al., MC Zeisel et al., HCA He et al.)
- Gene x phenotype discriminative score matrix M used by GreedyPC
- Operating point: number of markers to select (e.g., 42 genes for IPF)

## Outputs

- Selected marker set S per method and dataset
- macro_f1_results.csv with one macro-F1 per method-dataset pair
- Comparison against reported baselines with flagged deviations

## How to apply

Load the scRNA-seq count matrices (e.g., IPF lung from Adams et al., mouse cortex from Zeisel et al., human cell atlas from He et al.), filter genes/cells, normalize and log-transform. Compute phenotype-pair discriminative scores, then run GreedyPC from phenotype-cover at the reported operating point (e.g., 42 genes on IPF) alongside baseline selectors (scGeneFit, decision trees, mutual information via scikit-learn, TopDE, ReliefF, mrmr_classif from mrmr_selection) on identical splits. For each method, train a scikit-learn logistic regression on the same subset used for feature selection and evaluate macro-F1 on left-out cells — the paper's protocol is 'select a feature set S using a subset of the data, train a logistic regression model on the same subset, and evaluate performance on left-out data'. Compare reproduced scores against reported values (G-PC 0.70 on IPF with 42 genes, 0.94–0.95 on MC, 0.68 on HCA; decision trees 0.70 on IPF) and flag deviations. Write each method-dataset macro-F1 to macro_f1_results.csv.

## Related tools

- **phenotype-cover** (Provides GreedyPC and CEMPC for phenotype-cover marker selection; installed via pip install phenotype-cover) — https://github.com/euxhenh/phenotype-cover
- **multiset-multicover** (Dependency package for phenotype-cover; PC is shown to be equivalent to multiset multicover)
- **scikit-learn** (Logistic regression classifier for evaluation, plus decision-tree and mutual-information baseline selectors)
- **mrmr_selection** (mRMR baseline; use mrmr_classif(X=X, y=y, K=k) to select k categorical features on the same splits) — https://github.com/smazzanti/mrmr
- **numpy** (Preprocessing of count matrices (gene/cell filtering, normalization, log transform))

## Examples

```
from phenotype_cover import GreedyPC; gpc = GreedyPC(); gpc.fit(X, y); features = gpc.select(42)
```

## Evaluation signals

- Reproduced G-PC macro-F1 within tolerance of reported values: 0.70 on IPF with 42 genes, 0.94–0.95 on MC, 0.68 on HCA
- Decision-tree baseline reaches ~0.70 macro-F1 on IPF
- All methods evaluated on the same data subset used for feature selection and scored on left-out cells only
- Every method-dataset pair present as a row in macro_f1_results.csv

## Limitations

- G-PC is a polynomial-time approximation and does not provide an optimal solution to the NP-complete multiset multicover problem
- While G-PC worked well for the data analyzed, other coverage-optimizing approximation algorithms remain untested on biological data
- The biological analysis relied on overlap with existing gene marker lists that may be incomplete

## Evidence

- [results] G-PC vs baselines on IPF: [evidence span withheld — anchor status: not_found]
