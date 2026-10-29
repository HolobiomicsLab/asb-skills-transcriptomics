---
name: multi-dataset-scrnaseq-preprocessing
description: Use when raw scRNA-seq count matrices (e.g., IPF, MC, or HCA datasets)
  must be filtered, normalized, and log-transformed prior to running marker-selection
  methods such as GreedyPC or baseline selectors.
when_to_use_negative:
- Input is already a normalized or log-transformed expression matrix from an upstream
  pipeline — re-filtering could change the reported operating points (e.g., the 42-gene
  IPF set)
- Data are bulk RNA-seq or pseudo-bulk mixtures (e.g., the deconvolution pseudo-bulk
  mixtures averaged across test-set cells) rather than single-cell count matrices
- The analysis target is deconvolution (ν-SVR with CIBERSORT-style evaluation) where
  downstream steps expect the already-preprocessed matrices
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: numpy
  role: Implements gene/cell filtering, normalization, and log transformation of the
    scRNA-seq count matrices
  repo: https://github.com/euxhenh/phenotype-cover
- name: phenotype-cover
  role: Downstream consumer of the preprocessed matrices; its GreedyPC requires log-transformed
    X
  repo: https://github.com/euxhenh/phenotype-cover
- name: scikit-learn
  role: Dependency of phenotype-cover and basis for logistic regression evaluation
    on the preprocessed features
  repo: https://github.com/euxhenh/phenotype-cover
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100332
    title: Multiset multicover methods for discriminative marker selection
  source_content_hash: sha256:64591b5896a47fb44ea0ddc461f0fad7578ae2bd1dd53740cbb3ad7c3f481f11
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
  - numpy
  - phenotype-cover
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# multi-dataset-scrnaseq-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preprocesses the paper's three deposited scRNA-seq count matrices (IPF lung from Adams et al., mouse cortex from Zeisel et al., human cell atlas from He et al.) via gene/cell filtering, normalization, and log transformation with numpy, yielding the log-transformed expression matrices required before phenotype-pair discriminative scoring and marker selection.

## When to use

Use when raw scRNA-seq count matrices (e.g., IPF, MC, or HCA datasets) must be filtered, normalized, and log-transformed prior to running marker-selection methods such as GreedyPC or baseline selectors.

## When NOT to use

- Input is already a normalized or log-transformed expression matrix from an upstream pipeline — re-filtering could change the reported operating points (e.g., the 42-gene IPF set)
- Data are bulk RNA-seq or pseudo-bulk mixtures (e.g., the deconvolution pseudo-bulk mixtures averaged across test-set cells) rather than single-cell count matrices
- The analysis target is deconvolution (ν-SVR with CIBERSORT-style evaluation) where downstream steps expect the already-preprocessed matrices

## Inputs

- Deposited scRNA-seq count matrix for IPF lung (Adams et al.)
- Deposited scRNA-seq count matrix for mouse cortex (Zeisel et al.)
- Deposited scRNA-seq count matrix for human cell atlas (He et al.)
- Cell phenotype/cell-type labels per dataset

## Outputs

- Filtered, normalized, log-transformed expression matrix per dataset
- Preprocessed matrices aligned with phenotype labels for marker selection

## How to apply

Load each of the three deposited datasets (IPF lung - Adams et al., mouse cortex - Zeisel et al., human cell atlas - He et al.) and preprocess the count matrices in numpy by filtering genes/cells, normalizing, and log-transforming the values. Log transformation is needed because the phenotype-cover README notes raw counts may require it before fitting GreedyPC. Use identical preprocessing across all datasets and across G-PC and all baseline selectors (scGeneFit, decision trees, mutual information, TopDE, ReliefF, mRMR) so the same data splits can be compared fairly. Output normalized, log-transformed matrices that feed directly into phenotype-pair discriminative score computation for the PC problem. Retain the preprocessing parameters per dataset so reproduced macro-F1 results (e.g., G-PC 0.70 on IPF with 42 genes) can be traced back to identical inputs.

## Related tools

- **numpy** (Implements gene/cell filtering, normalization, and log transformation of the scRNA-seq count matrices) — https://github.com/euxhenh/phenotype-cover
- **phenotype-cover** (Downstream consumer of the preprocessed matrices; its GreedyPC requires log-transformed X) — https://github.com/euxhenh/phenotype-cover
- **scikit-learn** (Dependency of phenotype-cover and basis for logistic regression evaluation on the preprocessed features) — https://github.com/euxhenh/phenotype-cover

## Examples

```
from phenotype_cover import GreedyPC; import numpy as np; X = np.log1p(normalized_counts); y = cell_type_labels; gpc = GreedyPC(); gpc.fit(X, y); features = gpc.select(8)
```

## Evaluation signals

- Each dataset's expression matrix is filtered, normalized, and log-transformed before any marker-selection method runs
- Identical preprocessing and data splits are used for G-PC and all baselines (scGeneFit, decision trees, mutual information, TopDE, ReliefF, mRMR)
- Preprocessed matrices support computing phenotype-pair discriminative scores for every phenotype pair, per the PC formulation
- Reproduced macro-F1 values fall near reported numbers (G-PC 0.70 on IPF with 42 genes, 0.94-0.95 on MC, 0.68 on HCA); deviations are flagged
- Every method-dataset pair yields a macro-F1 entry in macro_f1_results.csv

## Limitations

- No detailed per-dataset filtering cutoffs (gene/cell thresholds) are specified in the article, so preprocessing parameters must be chosen and documented by the practitioner
- Different preprocessing choices can change the number of selected markers at the reported operating point (e.g., 42 genes on IPF) and shift macro-F1
- Biological interpretation depends on existing marker lists that may be incomplete, so preprocessing quality cannot be judged by marker overlap alone

## Evidence

- [other] Load the three deposited scRNA-seq datasets (IPF lung - Adams et al., mouse cortex - Zeisel et al., human cell atlas - He et al.) and preprocess the count matrices (gene/cell filtering, normalization, log transform) using numpy.: [evidence span withheld — anchor status: not_found]
- [other] Run the baseline selectors on identical data splits: [evidence span withheld — anchor status: not_found]
