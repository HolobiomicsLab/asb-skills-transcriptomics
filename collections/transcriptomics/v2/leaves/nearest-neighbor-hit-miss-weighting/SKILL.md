---
name: nearest-neighbor-hit-miss-weighting
description: Use when you need a per-feature ranking by discriminative power from
  a labeled scRNA-seq matrix (e.g. Zeisel et al. mouse cortex) to serve as a baseline
  against combinatorial marker-selection methods like phenotype cover.
when_to_use_negative:
- The goal is a globally discriminative marker set covering every phenotype pair (the
  phenotype cover problem) — ReliefF scores features individually and does not guarantee
  pairwise coverage; use GreedyPC/CEMPC instead.
- A minimal-optimal subset of features is required rather than a full ranking of all
  features (e.g. the mRMR setting described in the smazzanti/mrmr README).
- The data are raw counts that have not been preprocessed/log-transformed, so distances
  used for neighbor retrieval would be dominated by count scale.
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0203
tools:
- name: faiss
  role: Builds the approximate nearest-neighbor index over sample feature vectors
    and retrieves the 30 nearest neighbors per sample used for hit/miss contrasts
- name: numpy
  role: Holds the preprocessed expression matrix and labels and computes the per-feature
    ReliefF hit/miss weight arithmetic
- name: scikit-learn
  role: Supports the neighbor/label-based scoring operations in the ReliefF weight
    computation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100332
    title: Multiset multicover methods for discriminative marker selection
  source_content_hash: sha256:87b5d1f976d47570339fbf151bd4af29afe45524fd1e700425d75f4361e98e93
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
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# nearest-neighbor-hit-miss-weighting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

ReliefF-style per-feature weighting that retrieves 30 approximate nearest neighbors per sample with a faiss index and scores each feature by contrasting its values among same-label neighbors (hits) versus different-label neighbors (misses). Used in the marker-selection study as a neighbor-based baseline ranking against phenotype-cover methods (G-PC, CEM-PC).

## When to use

Use when you need a per-feature ranking by discriminative power from a labeled scRNA-seq matrix (e.g. Zeisel et al. mouse cortex) to serve as a baseline against combinatorial marker-selection methods like phenotype cover.

## When NOT to use

- The goal is a globally discriminative marker set covering every phenotype pair (the phenotype cover problem) — ReliefF scores features individually and does not guarantee pairwise coverage; use GreedyPC/CEMPC instead.
- A minimal-optimal subset of features is required rather than a full ranking of all features (e.g. the mRMR setting described in the smazzanti/mrmr README).
- The data are raw counts that have not been preprocessed/log-transformed, so distances used for neighbor retrieval would be dominated by count scale.

## Inputs

- Preprocessed scRNA-seq expression matrix as a numpy array (e.g. the mouse cortex / Zeisel et al. dataset used in the study)
- Phenotype/cell-type label vector aligned to matrix rows
- Neighbor count parameter k (30 nearest neighbors)

## Outputs

- relieff_faiss_ranked_features.csv with one row per feature containing a feature identifier and a numeric ReliefF score, sorted in descending order
- Ranked feature list usable for downstream top-K feature selection and baseline comparison

## How to apply

Load the preprocessed MC (Zeisel et al.) scRNA-seq expression matrix and phenotype labels as numpy arrays. Build a faiss approximate nearest-neighbor index over the sample feature vectors and, for each sample, query the index to retrieve its 30 nearest neighbors. Compute ReliefF weights using numpy/scikit-learn operations by contrasting each feature's value between each sample and its retrieved neighbors (hits, same phenotype) and samples of a different phenotype (misses); features that fail to separate an instance from its different-label neighbors are penalized, while features with similar values among same-class instances receive high scores. Sort features by weight in descending order and save one row per feature (identifier plus numeric score) to relieff_faiss_ranked_features.csv. The resulting ranking is a baseline whose selected feature set S can be evaluated by training a logistic regression model on a data subset and assessing performance on left-out data.

## Related tools

- **faiss** (Builds the approximate nearest-neighbor index over sample feature vectors and retrieves the 30 nearest neighbors per sample used for hit/miss contrasts)
- **numpy** (Holds the preprocessed expression matrix and labels and computes the per-feature ReliefF hit/miss weight arithmetic)
- **scikit-learn** (Supports the neighbor/label-based scoring operations in the ReliefF weight computation)

## Examples

```
import faiss; index = faiss.IndexFlatL2(X.shape[1]); index.add(X); D, I = index.search(X, 30)  # 30 nearest neighbors per sample for ReliefF hit/miss scoring
```

## Evaluation signals

- The output file relieff_faiss_ranked_features.csv exists and contains one row per feature with a numeric score column.
- Exactly 30 neighbors were retrieved for each sample from the faiss index.
- Scores are sorted in descending order and each feature's score reflects penalization when it fails to separate an instance from different-label (miss) neighbors.
- Features with similar values among same-class instances and large differences across phenotypes obtain the highest weights.
- Top-K features from the ranking yield competitive logistic-regression accuracy on left-out data and a stable stability index over 5 half-sampling runs.

## Limitations

- ReliefF is only one of the eight baseline feature-selection methods; the greedy algorithm (G-PC) was shown to outperform competitors across a variety of tasks.
- Neighbor retrieval relies on faiss approximate nearest neighbors and a fixed k of 30, so scores depend on approximation quality and the neighbor-count choice.
- Raw count matrices may require a log transform before neighbor-based distance computations are meaningful.

## Evidence
