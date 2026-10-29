---
name: relieff-feature-scoring
description: Use when you have a labeled expression matrix (samples x features with
  phenotype labels) and need an individual per-feature relevance score that reflects
  local neighborhood discrimination, e.g., as a baseline against phenotype-cover methods.
when_to_use_negative:
- When the goal is a minimal discriminative marker set covering all phenotype pairs
  rather than a per-feature relevance ranking — use phenotype-cover (G-PC/CEM-PC)
  instead
- When features must be evaluated jointly for redundancy (ReliefF scores each feature
  independently of other selected features)
tools:
- name: faiss
  role: Builds the approximate nearest neighbor index used to retrieve the 30 nearest
    neighbors for each sample during ReliefF weight computation
- name: numpy
  role: Array operations for loading the expression matrix and computing neighbor-based
    feature weights
- name: scikit-learn
  role: Supporting operations for class-aware neighbor (hit/miss) contrasts in the
    ReliefF computation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100332
    title: Multiset multicover methods for discriminative marker selection
  source_content_hash: sha256:6d14369a4cd4c275c59599c5d9c0fee8a1098927c4305071fe68d99f04bcc1fd
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/euxhenh/phenotype-cover
  tools:
  - faiss
  - numpy
  - scikit-learn
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# relieff-feature-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes ReliefF neighbor-based per-feature weights on a labeled scRNA-seq expression matrix (e.g., the MC mouse cortex dataset from Zeisel et al.), using faiss approximate nearest neighbors with k=30 to produce a ranked feature list usable as a baseline marker-selection method.

## When to use

Use when you have a labeled expression matrix (samples x features with phenotype labels) and need an individual per-feature relevance score that reflects local neighborhood discrimination, e.g., as a baseline against phenotype-cover methods.

## When NOT to use

- When the goal is a minimal discriminative marker set covering all phenotype pairs rather than a per-feature relevance ranking — use phenotype-cover (G-PC/CEM-PC) instead
- When features must be evaluated jointly for redundancy (ReliefF scores each feature independently of other selected features)

## Inputs

- Preprocessed scRNA-seq expression matrix as a numpy array (e.g., MC mouse cortex dataset, Zeisel et al.)
- Phenotype/class label array aligned to matrix rows (integer or string labels)

## Outputs

- relieff_faiss_ranked_features.csv: one row per feature with a feature identifier and a numeric ReliefF score column, sorted in descending score order

## How to apply

Load the preprocessed expression matrix and phenotype labels as numpy arrays. Build a faiss approximate nearest neighbor index over the sample feature vectors and, for each sample, query the index to retrieve its 30 nearest neighbors. Compute ReliefF weights by contrasting feature values between each sample and its retrieved neighbors (hits) versus samples of different phenotypes (misses) using numpy/scikit-learn operations; features that fail to separate an instance from different-label neighbors are penalized, while features with similar values among same-class instances receive high scores. Sort features by weight in descending order and save feature identifier plus numeric score per row to relieff_faiss_ranked_features.csv. This yields a univariate ranking that can be compared against multivariate marker-set methods such as G-PC/CEM-PC.

## Related tools

- **faiss** (Builds the approximate nearest neighbor index used to retrieve the 30 nearest neighbors for each sample during ReliefF weight computation)
- **numpy** (Array operations for loading the expression matrix and computing neighbor-based feature weights)
- **scikit-learn** (Supporting operations for class-aware neighbor (hit/miss) contrasts in the ReliefF computation)

## Examples

```
import faiss; index = faiss.IndexFlatL2(X.shape[1]); index.add(X); D, I = index.search(X, 30)  # then compute ReliefF hits/misses over I with labels y
```

## Evaluation signals

- Output file relieff_faiss_ranked_features.csv exists and contains one row per feature
- The output CSV has a numeric score column
- Exactly 30 neighbors were retrieved for each sample from the faiss index
- Features are sorted in descending order of ReliefF weight
- The feature identifiers in the output match those of the input expression matrix

## Limitations

- Neighbor-based scores rank features individually and do not guarantee a minimal or jointly discriminative marker set
- Results depend on the choice of 30 neighbors and on the quality of the approximate (not exact) nearest neighbor search
- Serves as one of eight baseline feature-selection methods in the study, not as the primary proposed approach

## Evidence

- [full_text] ReliefF neighbor scoring rationale: [evidence span withheld — anchor status: not_found]
