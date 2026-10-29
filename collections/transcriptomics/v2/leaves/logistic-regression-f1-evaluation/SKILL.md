---
name: logistic-regression-f1-evaluation
description: Use when a feature/marker set (e.g., from GreedyPC at a fixed coverage)
  needs downstream validation of how well it distinguishes phenotypes, via classification
  F1 on held-out samples from scRNA-seq data such as the IPF, MC, or HCA datasets.
when_to_use_negative:
- Input is a raw untransformed count matrix — log-transform first per the README before
  fitting
- The goal is deconvolution of bulk mixtures (pseudo-bulk via ν-SVR with ν in {0.25,
  0.5, 0.75} and RMSE), not classification of single-cell phenotypes
- You need an optimal marker set — G-PC is a polynomial-time approximation and does
  not guarantee an optimal solution
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scikit-learn
  role: Provides the logistic regression classifier used to train on the feature subset
    and measure F1 on left-out data
  repo: https://github.com/euxhenh/phenotype-cover
- name: GreedyPC
  role: Produces the marker set S whose discriminative quality is assessed by the
    logistic regression F1
  repo: https://github.com/euxhenh/phenotype-cover
- name: phenotype-cover
  role: Package supplying GreedyPC/CEMPC marker selection whose outputs feed the F1
    evaluation workflow
  repo: https://github.com/euxhenh/phenotype-cover
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100332
    title: Multiset multicover methods for discriminative marker selection
  source_content_hash: sha256:479a4dec3606012f3bed15bc94062b4270f1eeb7a874bf0aa8c46a80ff7279ca
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
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# Logistic Regression F1 Evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates a selected marker/feature set S by training a scikit-learn logistic regression model on a subset of the data and measuring F1 on left-out data, quantifying the discriminative power of markers chosen by phenotype-cover (G-PC/CEM-PC) selection.

## When to use

Use when a feature/marker set (e.g., from GreedyPC at a fixed coverage) needs downstream validation of how well it distinguishes phenotypes, via classification F1 on held-out samples from scRNA-seq data such as the IPF, MC, or HCA datasets.

## When NOT to use

- Input is a raw untransformed count matrix — log-transform first per the README before fitting
- The goal is deconvolution of bulk mixtures (pseudo-bulk via ν-SVR with ν in {0.25, 0.5, 0.75} and RMSE), not classification of single-cell phenotypes
- You need an optimal marker set — G-PC is a polynomial-time approximation and does not guarantee an optimal solution

## Inputs

- scRNA-seq expression matrix X (samples x features), log-transformed if raw counts
- Phenotype/cell-type labels y (e.g., n_classes=5 cell types)
- Selected marker set S (feature indices/names from GreedyPC or CEMPC)

## Outputs

- Held-out F1 score for the logistic regression trained on S
- Phenotype-pair coverage attained by S
- Results table (e.g., gpc_results.csv) with marker list, coverage, and F1 metrics

## How to apply

Select a feature set S using a subset of the data (e.g., markers output by GreedyPC.select(coverage)), train a scikit-learn logistic regression model on that same subset, and evaluate performance (F1) on left-out data to assess whether the markers accurately distinguish phenotypes. Report the phenotype-pair coverage achieved by S alongside the held-out F1 so marker compactness and discriminative power are compared together. Compare against the paper's reported coverage and F1 for the chosen dataset (e.g., mouse cortex) within a small tolerance as the acceptance criterion. If working with raw counts, log-transform X before fitting, as recommended by the phenotype-cover README.

## Related tools

- **scikit-learn** (Provides the logistic regression classifier used to train on the feature subset and measure F1 on left-out data) — https://github.com/euxhenh/phenotype-cover
- **GreedyPC** (Produces the marker set S whose discriminative quality is assessed by the logistic regression F1) — https://github.com/euxhenh/phenotype-cover
- **phenotype-cover** (Package supplying GreedyPC/CEMPC marker selection whose outputs feed the F1 evaluation workflow) — https://github.com/euxhenh/phenotype-cover

## Examples

```
from sklearn.linear_model import LogisticRegression; from sklearn.metrics import f1_score; clf = LogisticRegression().fit(X_train[:, features], y_train); f1 = f1_score(y_test, clf.predict(X_test[:, features]), average='macro')
```

## Evaluation signals

- Held-out F1 on left-out data matches the paper's reported performance for the chosen dataset within a small tolerance
- The marker set S attains the paper's reported phenotype-pair coverage on the chosen dataset
- F1 is computed from a model trained only on the subset used for feature selection, evaluated strictly on left-out samples
- Coverage reported alongside F1 is consistent with the coverage parameter used in GreedyPC.select(coverage)

## Limitations

- G-PC does not provide an optimal solution, so F1 and coverage may be suboptimal
- Biological comparison relied on overlapping with existing gene marker lists that may be incomplete
- No more thorough biological analysis of the selected genes' relation to each cell type was performed

## Evidence
