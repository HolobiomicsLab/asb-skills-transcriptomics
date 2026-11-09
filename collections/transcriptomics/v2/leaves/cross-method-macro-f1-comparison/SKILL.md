---
name: cross-method-macro-f1-comparison
description: 'Use when comparing discriminative power of marker-selection methods
  on scRNA-seq phenotype classification: you need each selector''s features (e.g.,
  42 genes on IPF) scored identically via logistic regression macro-F1.'
when_to_use_negative:
- Input is already a ranked feature list without associated expression data — no logistic
  regression can be trained/evaluated on left-out cells
- The task is single-pair differential expression ranking (TopDE-style) rather than
  selecting a global marker set discriminative across all phenotype pairs
- Deconvolution of bulk mixtures or stability testing is the goal — those use pseudo-bulk
  ν-SVR or half-sampling stability indices, not macro-F1 classification
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3474
tools:
- name: GreedyPC
  role: Primary marker selector; run via phenotype-cover at the reported operating
    point (42 genes on IPF) before classifier evaluation
  repo: https://github.com/euxhenh/phenotype-cover
- name: CEMPC
  role: Cross-entropy-method baseline selector (CEM-PC) from the same phenotype-cover
    package
  repo: https://github.com/euxhenh/phenotype-cover
- name: scikit-learn
  role: Trains logistic regression on each selector's features, evaluates macro-F1
    on left-out cells, and provides decision-tree and mutual-information baseline
    selectors
- name: mRMR
  role: Baseline minimal-optimal feature selector via mrmr_classif on the same data
    splits
  repo: https://github.com/smazzanti/mrmr
- name: numpy
  role: Preprocessing of count matrices (gene/cell filtering, normalization, log transform)
    and score matrix construction
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100332
    title: Multiset multicover methods for discriminative marker selection
  source_content_hash: sha256:813b7a18587046ed02d8290db06aecc6a4869fed6c8fed14f8bf5a5af7126508
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
  - CEMPC
  - GreedyPC
  - Python
  - mRMR
  - numpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# cross-method-macro-f1-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark protocol comparing G-PC marker selection against baselines (scGeneFit, decision trees, mutual information, TopDE, ReliefF, mRMR, CEM-PC) by training logistic regression on each method's selected features and reporting macro-F1 on left-out cells across the IPF, mouse cortex (MC), and human cell atlas (HCA) scRNA-seq datasets.

## When to use

Use when comparing discriminative power of marker-selection methods on scRNA-seq phenotype classification: you need each selector's features (e.g., 42 genes on IPF) scored identically via logistic regression macro-F1.

## When NOT to use

- Input is already a ranked feature list without associated expression data — no logistic regression can be trained/evaluated on left-out cells
- The task is single-pair differential expression ranking (TopDE-style) rather than selecting a global marker set discriminative across all phenotype pairs
- Deconvolution of bulk mixtures or stability testing is the goal — those use pseudo-bulk ν-SVR or half-sampling stability indices, not macro-F1 classification

## Inputs

- Preprocessed, log-transformed scRNA-seq count matrices for IPF (Adams et al.), mouse cortex (Zeisel et al.), and HCA (He et al.)
- Phenotype-pair discriminative score matrix M (phenotype x feature) for GreedyPC/CEM-PC
- Feature sets from baseline selectors: scGeneFit, decision trees, mutual information, TopDE, ReliefF, mRMR

## Outputs

- macro_f1_results.csv with macro-F1 for every method-dataset pair
- Selected gene lists per method per dataset (e.g., 42 genes on IPF for G-PC)
- Deviation flags comparing reproduced vs. reported scores (G-PC 0.70 IPF, 0.94-0.95 MC, 0.68 HCA)

## How to apply

Load and preprocess the three scRNA-seq datasets (IPF lung - Adams et al., mouse cortex - Zeisel et al., human cell atlas - He et al.) with gene/cell filtering, normalization, and log transform in numpy. On each dataset, compute phenotype-pair discriminative scores and run GreedyPC from phenotype-cover at the reported operating point (e.g., 42 genes on IPF), then run all baseline selectors (scGeneFit, scikit-learn decision trees and mutual-information ranking, TopDE, ReliefF, mRMR from smazzanti/mrmr) on identical data splits. For each method, train a scikit-learn logistic regression on the same data subset used for feature selection and evaluate macro-F1 on left-out cells, mirroring the paper's protocol of selecting a feature set S on a subset, training on that subset, and evaluating on left-out data. Compare reproduced scores against reported values (G-PC 0.70 on IPF with 42 genes, 0.94-0.95 on MC, 0.68 on HCA; decision trees 0.70 on IPF) and flag deviations. Write every method-dataset macro-F1 to macro_f1_results.csv.

## Related tools

- **GreedyPC** (Primary marker selector; run via phenotype-cover at the reported operating point (42 genes on IPF) before classifier evaluation) — https://github.com/euxhenh/phenotype-cover
- **CEMPC** (Cross-entropy-method baseline selector (CEM-PC) from the same phenotype-cover package) — https://github.com/euxhenh/phenotype-cover
- **scikit-learn** (Trains logistic regression on each selector's features, evaluates macro-F1 on left-out cells, and provides decision-tree and mutual-information baseline selectors)
- **mRMR** (Baseline minimal-optimal feature selector via mrmr_classif on the same data splits) — https://github.com/smazzanti/mrmr
- **numpy** (Preprocessing of count matrices (gene/cell filtering, normalization, log transform) and score matrix construction)

## Examples

```
from phenotype_cover import GreedyPC; gpc = GreedyPC(); gpc.fit(X_train, y_train); S = gpc.select(8); clf = LogisticRegression().fit(X_train[:, S], y_train); print(f1_score(y_test, clf.predict(X_test[:, S]), average='macro'))
```

## Evaluation signals

- G-PC reproduces macro-F1 0.70 on IPF with 42 selected genes, with scGeneFit at 0.65 and CEM-PC at 0.61 below it
- G-PC macro-F1 falls in 0.94-0.95 on the mouse cortex dataset and 0.68 on the HCA dataset
- Decision-tree baseline reproduces 0.70 on IPF, matching the reported tie with G-PC
- Every method-dataset pair present in macro_f1_results.csv with a numeric macro-F1; identical data splits used for selection and classifier training across methods

## Limitations

- G-PC does not provide an optimal solution; reproduced scores may deviate from reported values depending on data splits
- While G-PC worked well for the data analyzed in this paper, its greedy approximation may not generalize to all marker-selection tasks
- Baselines beyond scGeneFit, decision trees, MI, TopDE, ReliefF, and mRMR were not included; other coverage-optimizing approximation algorithms remain untested on biological data

## Evidence

- [other] reported IPF ranking: [evidence span withheld — anchor status: not_found]
- [readme] mRMR categorical selection: [evidence span withheld — anchor status: not_found]
