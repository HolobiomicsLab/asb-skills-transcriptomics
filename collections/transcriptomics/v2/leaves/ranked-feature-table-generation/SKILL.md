---
name: ranked-feature-table-generation
description: Use when you have a labeled feature matrix (e.g. scRNA-seq expression
  with phenotype labels) and need a per-feature numeric score to produce a full descending
  feature ranking, as a baseline filter before or alongside marker-set methods like
  GreedyPC.
when_to_use_negative:
- The goal is a small globally discriminative marker set covering all phenotype pairs
  — use the phenotype cover algorithms (GreedyPC or CEM-PC) instead of a per-feature
  ranking filter
- No phenotype labels are available, since ReliefF requires contrasting same-class
  neighbors against different-label samples
- Input is already a ranked or selected feature list rather than a raw feature-by-sample
  matrix
tools:
- name: faiss
  role: Builds the approximate nearest neighbor index and retrieves the 30 neighbors
    per sample used as ReliefF hits/misses
- name: numpy
  role: Holds the expression matrix and labels and performs the vectorized ReliefF
    weight computation
- name: scikit-learn
  role: Used alongside numpy for the per-feature scoring operations in the ReliefF
    weight calculation
- name: mrmr
  role: Alternative ranking filter whose output is likewise a ranked feature list
    from which the first K elements are selected
  repo: smazzanti/mrmr
- name: phenotype-cover
  role: Companion marker-selection method whose GreedyPC.select(K) consumes/produces
    ranked feature choices; baseline rankings can be compared against its solution
  repo: euxhenh/phenotype-cover
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100332
    title: Multiset multicover methods for discriminative marker selection
  source_content_hash: sha256:80c00097eece8c246e4178243f874ce6eda6fc8566e83ef23e35d11f98a8207b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/euxhenh/phenotype-cover
  repo_url: https://github.com/smazzanti/mrmr
  tools:
  - faiss
  - mrmr
  - numpy
  - phenotype-cover
  - scikit-learn
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# ranked-feature-table-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates a per-feature ranked table (e.g. relieff_faiss_ranked_features.csv) by computing a numeric importance score for every feature — in the marker-selection study, ReliefF weights over the Mouse Cortex (Zeisel et al.) scRNA-seq matrix using 30 faiss approximate-nearest-neighbor hits versus different-phenotype misses.

## When to use

Use when you have a labeled feature matrix (e.g. scRNA-seq expression with phenotype labels) and need a per-feature numeric score to produce a full descending feature ranking, as a baseline filter before or alongside marker-set methods like GreedyPC.

## When NOT to use

- The goal is a small globally discriminative marker set covering all phenotype pairs — use the phenotype cover algorithms (GreedyPC or CEM-PC) instead of a per-feature ranking filter
- No phenotype labels are available, since ReliefF requires contrasting same-class neighbors against different-label samples
- Input is already a ranked or selected feature list rather than a raw feature-by-sample matrix

## Inputs

- Preprocessed scRNA-seq expression matrix (numpy array), e.g. Mouse Cortex (MC) dataset of Zeisel et al.
- Per-sample phenotype/class labels aligned with matrix rows
- Neighbor count parameter (30 nearest neighbors)

## Outputs

- relieff_faiss_ranked_features.csv with one row per feature: feature identifier plus numeric ReliefF score, sorted in descending score order

## How to apply

Load the preprocessed MC (Zeisel et al.) scRNA-seq expression matrix and phenotype labels as numpy arrays, then build a faiss approximate nearest neighbor index over the sample vectors and query it to retrieve 30 nearest neighbors per sample. Compute ReliefF weights by contrasting each sample's feature values with same-class neighbors (hits) and samples of different phenotypes (misses) using numpy/scikit-learn operations; features are penalized when they fail to distinguish an instance from different-label neighbors and score high when values are similar among same-class instances. Sort features by weight in descending order and write one row per feature with an identifier and numeric score to relieff_faiss_ranked_features.csv. As in the paper's ranking-based baselines, treat the output as a ranking: taking the first k elements yields a k-feature subset, analogous to mrmr_classif's ranked list output.

## Related tools

- **faiss** (Builds the approximate nearest neighbor index and retrieves the 30 neighbors per sample used as ReliefF hits/misses)
- **numpy** (Holds the expression matrix and labels and performs the vectorized ReliefF weight computation)
- **scikit-learn** (Used alongside numpy for the per-feature scoring operations in the ReliefF weight calculation)
- **mrmr** (Alternative ranking filter whose output is likewise a ranked feature list from which the first K elements are selected) — smazzanti/mrmr
- **phenotype-cover** (Companion marker-selection method whose GreedyPC.select(K) consumes/produces ranked feature choices; baseline rankings can be compared against its solution) — euxhenh/phenotype-cover

## Examples

```
from mrmr import mrmr_classif; selected_features = mrmr_classif(X=X, y=y, K=10)  # ranked list; take first elements for further selection
```

## Evaluation signals

- The output file relieff_faiss_ranked_features.csv exists and contains exactly one row per feature with a numeric score column
- Verification confirms 30 neighbors were retrieved for each sample from the faiss index
- Scores are sorted in strictly descending order so the table is a valid ranking
- Top-ranked features separate same-class instances (high similarity among class neighbors) while low-scoring features fail to distinguish different-label neighbors
- Subsetting the top k features yields features usable in downstream logistic regression train/test evaluation as done for baseline methods in the study

## Limitations

- Scores are neighbor-based and depend on the approximate nearest neighbor search, so approximate (not exact) neighbors can affect weights
- ReliefF is a filter ranking per feature and, unlike the phenotype cover formulation, does not guarantee a global set distinguishing every phenotype pair
- In multiorgan datasets, an overall DE-style ranking may miss markers specific to both a cell type and a tissue, which is precisely the case where PC-based selection was motivated

## Evidence

- [other] ReliefF penalizes features that fail to distinguish an instance from its negative (different-label) neighbors and assigns high scores to features with similar values among instances of the same class: [evidence span withheld — anchor status: not_found]
- [other] For each sample, query the index to retrieve its 30 nearest neighbors.: [evidence span withheld — anchor status: not_found]
- [readme] Note: the output of mrmr_classif is a list containing K selected features. This is a ranking, therefore, if you want to make a further selection, take the first elements of this list.: [evidence span withheld — anchor status: not_found]
