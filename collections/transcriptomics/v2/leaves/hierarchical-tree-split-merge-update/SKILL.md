---
name: hierarchical-tree-split-merge-update
description: Use when a new labeled scRNA-seq dataset (AnnData with cell population
  labels and batch indicators) must be progressively integrated into an existing scHPL
  classification tree, and you need to determine which populations match, merge, split,
  or are new.
when_to_use_negative:
- Datasets are not yet aligned in a shared latent space — scHPL is not a batch correction
  tool, so matching must not be run on raw counts
- You only need to predict labels of an unlabeled dataset using an existing tree (use
  scHPL's predict mode instead of progressive tree updating)
- Cell-type relationships are known to violate a tree structure, e.g. dedifferentiation
  such as beta-to-alpha cell conversions in type 2 diabetes
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scHPL
  role: 'Implements the hierarchical progressive learning: trains per-node classifiers
    (linear SVM, kNN, or one-class SVM), computes matching from cross-predictions,
    and updates the classification tree'
  repo: https://github.com/lcmmichielsen/scHPL
- name: scArches
  role: Recommended for aligning datasets into a shared latent space before cell-population
    matching, via the treeArches framework
  repo: https://github.com/theislab/scarches
- name: scVI
  role: Alternative to scArches for integrating datasets prior to running scHPL tree
    updates
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-23196-8
    title: Hierarchical progressive learning of cell identities in single-cell data
  source_content_hash: sha256:29decca3ba86c1180856e8d5946f1f5fb8dee2902cd443e7b4a23e516e14ae34
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lcmmichielsen/scHPL
  tools:
  - scArches
  - scHPL
  - scVI
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# hierarchical-tree-split-merge-update

## Summary

Reconstructs scHPL's reciprocal label-matching and tree-updating procedure: cross-predict populations between two aligned datasets, build a matching matrix from binarized confusion matrices, and update the hierarchical classification tree via match, merge, split, or new-population operations.

## When to use

Use when a new labeled scRNA-seq dataset (AnnData with cell population labels and batch indicators) must be progressively integrated into an existing scHPL classification tree, and you need to determine which populations match, merge, split, or are new.

## When NOT to use

- Datasets are not yet aligned in a shared latent space — scHPL is not a batch correction tool, so matching must not be run on raw counts
- You only need to predict labels of an unlabeled dataset using an existing tree (use scHPL's predict mode instead of progressive tree updating)
- Cell-type relationships are known to violate a tree structure, e.g. dedifferentiation such as beta-to-alpha cell conversions in type 2 diabetes

## Inputs

- Aligned, labeled AnnData objects for datasets D1 and D2 (labels and batch indicators stored as metadata columns)
- Pre-built hierarchical classification tree for D1 (from scHPL's learn module)
- Cross-prediction confusion matrices C1 and C2 between D1 and D2 populations

## Outputs

- Matching matrix X identifying one-to-one matches, merges, splits, and unmatched populations
- Updated hierarchical classification tree persisted as learned_tree.pkl
- Binarized confusion matrices BC1 and BC2 (thresholded at fraction rank 0.25)

## How to apply

Load the aligned, labeled AnnData objects for datasets D1 and D2 (align first with scVI or scArches, since scHPL is not a batch correction tool). Train the hierarchical classifiers on the tree of D1 and cross-predict labels of D2 to obtain confusion matrices C1 and C2. Normalize the confusion matrices to fractions per population and binarize them (BC1, BC2) using a fraction-ranking threshold of 0.25 to decide which population pairs are matched. Compute the matching matrix X as the sum of the transpose of BC1 and BC2; X identifies one-to-one matches, merges, splits, and unmatched populations. Update the D1 tree accordingly: attach unmatched D2 populations as new nodes, merge split populations under a shared parent, and relabel matched nodes; persist the tree with scHPL's learn module to learned_tree.pkl.

## Related tools

- **scHPL** (Implements the hierarchical progressive learning: trains per-node classifiers (linear SVM, kNN, or one-class SVM), computes matching from cross-predictions, and updates the classification tree) — https://github.com/lcmmichielsen/scHPL
- **scArches** (Recommended for aligning datasets into a shared latent space before cell-population matching, via the treeArches framework) — https://github.com/theislab/scarches
- **scVI** (Alternative to scArches for integrating datasets prior to running scHPL tree updates)

## Examples

```
from scHPL import learn; updated_tree = learn(tree, data_new, reorder=False)
```

## Evaluation signals

- learned_tree.pkl exists and can be reloaded with scHPL without errors
- Every D2 population is either matched to a D1 node or attached as a new node after reloading the saved tree
- Matched population pairs are reciprocal (present in both BC1 and BC2 above the 0.25 fraction threshold), while merges/splits appear as non-square blocks in X
- Split populations end up merged under a shared parent in the updated tree
- Original annotations of D1 are preserved where populations matched perfectly (no relabeling of matched nodes beyond cross-dataset annotation)

## Limitations

- When a new dataset is added, the alignment — and consequently scHPL — has to be rerun, since current batch-correction workflows do not support incremental alignment
- Batch effects make it hard to resolve whether tree mistakes are caused by erroneous alignment or by mismatches created by scHPL
- Performance (especially of the one-class SVM) decreases on populations with a small number of cells
- A tree is not adequate in situations involving dedifferentiation, e.g. beta to alpha cell conversions in type 2 diabetes

## Evidence

- [methods] scHPL is a method that automatically finds relationships between cell populations across multiple datasets and uses this to construct a hierarchical classification tree.: "scHPL is a method that automatically finds relationships between cell populations across multiple datasets and uses this to construct a hierarchical classification tree."
- [methods] For each node in the tree either a linear SVM, kNN, or one-class SVM is trained.: "For each node in the tree either a linear SVM, kNN, or one-class SVM is trained."
- [readme] scHPL is not a batch correction tool, we advise to align the datasets before matching the cell populations.: "NOTE: scHPL is not a batch correction tool, we advise to align the datasets before matching the cell populations."
- [discussion] Since batch effects are inevitable when combining datasets, we require datasets to be aligned before running scHPL: "Since batch effects are inevitable when combining datasets, we require datasets to be aligned before running scHPL"
- [readme] treeArches is a framework around scHPL and scArches to automatically build and update reference atlases and the classification tree.: "treeArches is a framework around scHPL and `scArches <https://github.com/theislab/scarches>`_ to automatically build and update reference atlases and the classification tree."
