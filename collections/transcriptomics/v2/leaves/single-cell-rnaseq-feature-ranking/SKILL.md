---
name: single-cell-rnaseq-feature-ranking
description: Use when you have a preprocessed scRNA-seq expression matrix (e.g., the
  Mouse cortex MC/Zeisel et al. dataset) with phenotype labels and need a per-gene
  score to rank all features before selecting a discriminative marker set.
when_to_use_negative:
- Goal is a minimal subset of features that collectively distinguishes every phenotype
  pair (phenotype cover problem) — use GreedyPC or CEMPC from phenotype-cover instead
  of a per-feature ranking
- Input is already a ranked or selected feature list, or a phenotype x feature score
  matrix M ready for cover optimization
- The target variable is numeric regression rather than categorical phenotype labels
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: numpy
  role: Holds the preprocessed expression matrix and labels; used to compute the neighbor-contrast
    ReliefF weights and to sort features by score
- name: scikit-learn
  role: Supports the per-feature scoring operations used when contrasting samples
    against same-class neighbors and different-class samples
- name: faiss
  role: Builds the approximate nearest neighbor index over sample feature vectors
    and retrieves the 30 nearest neighbors per sample
- name: phenotype-cover (GreedyPC/CEMPC)
  role: 'Downstream consumer of ranked features: solves the phenotype cover problem
    over a phenotype x feature score matrix to pick a discriminative marker set'
  repo: https://github.com/euxhenh/phenotype-cover
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100332
    title: Multiset multicover methods for discriminative marker selection
  source_content_hash: sha256:c4d45701c98cc00d8cf669848a62d5413098e31460f7584f24e1c83f1233e0ed
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
  - faiss
  - numpy
  - phenotype-cover (GreedyPC/CEMPC)
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# single-cell-rnaseq-feature-ranking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Ranks individual genes of a preprocessed single-cell RNA-seq expression matrix by their per-feature discriminative scores, here via ReliefF neighbor-based feature weighting with faiss approximate nearest neighbors. Provides a ranked feature list that serves as input to marker-set methods such as the phenotype cover (PC) formulation.

## When to use

Use when you have a preprocessed scRNA-seq expression matrix (e.g., the Mouse cortex MC/Zeisel et al. dataset) with phenotype labels and need a per-gene score to rank all features before selecting a discriminative marker set.

## When NOT to use

- Goal is a minimal subset of features that collectively distinguishes every phenotype pair (phenotype cover problem) — use GreedyPC or CEMPC from phenotype-cover instead of a per-feature ranking
- Input is already a ranked or selected feature list, or a phenotype x feature score matrix M ready for cover optimization
- The target variable is numeric regression rather than categorical phenotype labels

## Inputs

- Preprocessed single-cell RNA-seq expression matrix as a numpy array (e.g., MC/Zeisel et al. mouse cortex dataset)
- Vector of phenotype/cell-type labels aligned with the matrix rows (number of neighbors k=30)

## Outputs

- Ranked feature list CSV (e.g., relieff_faiss_ranked_features.csv) with one row per gene containing a feature identifier and a numeric ReliefF score, sorted in descending score order

## How to apply

Load the preprocessed expression matrix and phenotype labels as numpy arrays (e.g., the MC scRNA-seq data from Zeisel et al.). Build a faiss approximate nearest neighbor index over the sample feature vectors and query it to retrieve the 30 nearest neighbors per sample. Compute ReliefF weights by contrasting each feature's values between a sample and its retrieved neighbors (hits of the same class) versus samples of different phenotypes (misses): features that fail to distinguish an instance from different-label neighbors are penalized, while features with similar values among same-class instances receive high scores. Sort features by weight in descending order to produce the ranked list and save it with one feature identifier and numeric score per row (e.g., relieff_faiss_ranked_features.csv). This ranking is a baseline complement to subset-selection methods like mRMR and the phenotype-cover solvers G-PC and CEM-PC, which operate on a phenotype x feature score matrix M.

## Related tools

- **numpy** (Holds the preprocessed expression matrix and labels; used to compute the neighbor-contrast ReliefF weights and to sort features by score)
- **scikit-learn** (Supports the per-feature scoring operations used when contrasting samples against same-class neighbors and different-class samples)
- **faiss** (Builds the approximate nearest neighbor index over sample feature vectors and retrieves the 30 nearest neighbors per sample)
- **phenotype-cover (GreedyPC/CEMPC)** (Downstream consumer of ranked features: solves the phenotype cover problem over a phenotype x feature score matrix to pick a discriminative marker set) — https://github.com/euxhenh/phenotype-cover

## Examples

```
import faiss, numpy as np; index = faiss.IndexFlatL2(X.shape[1]); index.add(X.astype('float32')); D, I = index.search(X.astype('float32'), 30)
```

## Evaluation signals

- Output file relieff_faiss_ranked_features.csv exists and contains one row per feature with a numeric score column
- 30 neighbors were successfully retrieved for every sample from the faiss index (no failed queries or short neighbor lists)
- Feature scores are sorted in strictly descending order, and the top-ranked genes are biologically plausible cell-type markers for the MC dataset
- Ranked scores differentiate phenotype classes: high-scoring features show similar values within a class and distinct values across classes
- Downstream, training a logistic regression on the top-ranked feature subset and evaluating on left-out data yields discriminative performance comparable to other baseline selection methods

## Limitations

- ReliefF is a per-feature (all-relevant style) ranking; it does not directly handle redundancy between features, unlike minimal-optimal methods such as mRMR or the multiset-multicover-based G-PC/CEM-PC solutions
- Neighbor quality depends on the faiss approximate index — approximation errors can degrade the hit/miss contrast, and k=30 is a fixed choice tied to this workflow
- Raw counts may need log-transformation before neighbor search (README notes: 'You may need to log-transform X if working with raw counts')

## Evidence

- [methods] ReliefF penalizes features that fail to distinguish an instance from its negative (different-label) neighbors and assigns high scores to features with similar values among instances of the same class: [evidence span withheld — anchor status: not_found]
- [methods] Build a faiss approximate nearest neighbor index over the sample feature vectors; query the index to retrieve its 30 nearest neighbors.: [evidence span withheld — anchor status: not_found]
- [methods] Sort features by their ReliefF weights in descending order to produce the ranked feature list.: [evidence span withheld — anchor status: not_found]
