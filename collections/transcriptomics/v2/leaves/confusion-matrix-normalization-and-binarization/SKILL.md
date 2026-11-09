---
name: confusion-matrix-normalization-and-binarization
description: Use when two aligned, labeled scRNA-seq datasets (AnnData with label
  and batch metadata) have been cross-predicted by scHPL classifiers and you must
  decide which population labels match, merge, split, or are new before updating the
  classification tree.
when_to_use_negative:
- Datasets are not yet aligned/integrated — scHPL is not a batch correction tool and
  requires alignment (e.g. with scVI or scArches) before matching cell populations
- You have only a single labeled dataset with no second dataset to cross-predict against,
  so no confusion matrices C1/C2 can be formed
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scHPL
  role: Provides the hierarchical progressive-learning framework whose learn module
    normalizes/binarizes the cross-prediction confusion matrices, computes the matching
    matrix, and updates the classification tree
  repo: https://github.com/lcmmichielsen/scHPL
- name: scArches
  role: Used upstream to align/integrate the two datasets before label matching, since
    scHPL is not a batch correction tool
  repo: https://github.com/theislab/scarches
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-23196-8
    title: Hierarchical progressive learning of cell identities in single-cell data
  source_content_hash: sha256:97b5630d09eded6c9f469245ce27827d323a8dc8a517f8f9858f528df88ffa16
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lcmmichielsen/scHPL
  tools:
  - scArches
  - scHPL
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# Confusion-matrix normalization and binarization for label matching

## Summary

scHPL's reciprocal label-matching step: confusion matrices from cross-predictions between two datasets (C1, C2) are normalized to per-population fractions, binarized with a fraction-ranking threshold of 0.25 into BC1/BC2, and summed (as BC1^T + BC2) into a matching matrix X that drives tree updates.

## When to use

Use when two aligned, labeled scRNA-seq datasets (AnnData with label and batch metadata) have been cross-predicted by scHPL classifiers and you must decide which population labels match, merge, split, or are new before updating the classification tree.

## When NOT to use

- Datasets are not yet aligned/integrated — scHPL is not a batch correction tool and requires alignment (e.g. with scVI or scArches) before matching cell populations
- You have only a single labeled dataset with no second dataset to cross-predict against, so no confusion matrices C1/C2 can be formed

## Inputs

- Aligned, labeled AnnData objects for datasets D1 and D2 with labels and batch indicators as metadata columns (post scVI/scArches integration)
- Cross-prediction confusion matrices C1 and C2 between the two datasets
- Hierarchical classification tree trained on D1

## Outputs

- Normalized confusion matrices (per-population fractions)
- Binarized confusion matrices BC1 and BC2 (threshold 0.25 fraction-ranking)
- Matching matrix X = BC1^T + BC2 encoding matches, merges, splits, and unmatched populations
- Updated hierarchical classification tree persisted as learned_tree.pkl

## How to apply

Train the hierarchical classifiers on the tree of dataset D1 and predict labels of dataset D2 (and vice versa) to obtain the cross-prediction confusion matrices C1 and C2. Normalize each confusion matrix to fractions per population, then binarize them into BC1 and BC2 using a fraction-ranking threshold of 0.25 to decide which population pairs are matched. Compute the matching matrix X as the sum of the transpose of BC1 and BC2; entries of X reveal one-to-one matches, merges, splits, and populations without a match. Use X to update the D1 tree: attach unmatched D2 populations as new populations, merge split populations under a shared parent, and relabel matched nodes. The rationale is that reciprocal (symmetrized) high-fraction cross-predictions are the evidence that two labels across datasets refer to the same cell population. Persist the resulting tree with scHPL's learn module to learned_tree.pkl.

## Related tools

- **scHPL** (Provides the hierarchical progressive-learning framework whose learn module normalizes/binarizes the cross-prediction confusion matrices, computes the matching matrix, and updates the classification tree) — https://github.com/lcmmichielsen/scHPL
- **scArches** (Used upstream to align/integrate the two datasets before label matching, since scHPL is not a batch correction tool) — https://github.com/theislab/scarches

## Examples

```
X = BC1.T + BC2  # matching matrix from binarized (threshold 0.25) cross-prediction confusion matrices, then update tree with scHPL's learn module -> learned_tree.pkl
```

## Evaluation signals

- learned_tree.pkl exists and can be reloaded with scHPL's learn module
- Reloaded tree reflects the split/merge/new-population updates implied by X
- Every D2 population is either matched to a D1 node in the tree or attached as a new node
- Binarized matrices BC1/BC2 are consistent with the 0.25 fraction-ranking threshold applied to the normalized confusion matrices
- Matching matrix X correctly distinguishes one-to-one matches from merge/split patterns (BC1^T + BC2 structure)

## Limitations

- If datasets are misaligned, it is hard to resolve whether tree mistakes stem from erroneous alignment or mismatches created by scHPL
- Adding a new dataset requires rerunning the alignment and consequently scHPL
- The hierarchical-relationship assumption fails for situations like dedifferentiation (e.g. beta to alpha cell conversions in type 2 diabetes), and a constant terminology for cell populations across datasets is missing
- Classifier performance decreases on populations with a small number of cells, which can affect the confusion matrices feeding the matching step

## Evidence

- [other] scHPL constructs the matching matrix X from the confusion matrices of the cross-predictions between two datasets, and then updates the classification tree based on X: [evidence span withheld — anchor status: not_found]
- [methods] scHPL automatically finds relationships between cell populations across multiple datasets: "scHPL is a method that automatically finds relationships between cell populations across multiple datasets and uses this to construct a hierarchical classification tree."
- [readme] Datasets must be aligned before running scHPL because it is not a batch correction tool: "NOTE: scHPL is not a batch correction tool, we advise to align the datasets before matching the cell populations. We advise doing this with scVI or scArches"
- [discussion] Batch effects require alignment before running scHPL: "Since batch effects are inevitable when combining datasets, we require datasets to be aligned before running scHPL"
