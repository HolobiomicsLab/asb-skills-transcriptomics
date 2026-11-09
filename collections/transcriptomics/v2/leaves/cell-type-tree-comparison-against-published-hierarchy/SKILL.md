---
name: cell-type-tree-comparison-against-published-hierarchy
description: Use when a learned or predicted cell-type hierarchy must be verified
  against a published cluster-correspondence figure, e.g. checking that AMB2016/AMB2018
  visual cortex populations attach to the correct parents.
when_to_use_negative:
- Datasets are not yet aligned — scHPL is not a batch correction tool, so running
  tree construction/comparison on unaligned data makes it impossible to tell whether
  tree mistakes come from erroneous alignment or from scHPL mismatches
- The cell populations of interest are absent from both the learned tree and any published
  reference (e.g. many brain cell populations are missing from the Cell Ontology),
  leaving nothing to compare against
tools:
- name: scHPL
  role: Constructs the hierarchical classification tree via scHPL.learn on the two
    aligned datasets and trains per-node classifiers (linear SVM) whose tree is compared
    to the published hierarchy
  repo: https://github.com/lcmmichielsen/scHPL
- name: Seurat
  role: Used to align the AMB2016 and AMB2018 datasets before scHPL matches cell populations,
    producing the integrated .X matrix fed into tree learning
- name: scikit-learn
  role: Provides the linear SVM trained at each node of the learned tree
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-23196-8
    title: Hierarchical progressive learning of cell identities in single-cell data
  source_content_hash: sha256:0978b57ec93ee39b6d6b97e41bba4edf33cce1f7b4e100fbfff87844d404d184
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lcmmichielsen/scHPL
  tools:
  - Seurat
  - scHPL
  - scikit-learn
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# cell-type-tree-comparison-against-published-hierarchy

## Summary

Validates a scHPL-constructed hierarchical classification tree of brain cell populations by comparing its learned relationships to a published reference hierarchy (Tasic et al. Extended Data Fig. 6), checking specific parent-child attachments and merges.

## When to use

Use when a learned or predicted cell-type hierarchy must be verified against a published cluster-correspondence figure, e.g. checking that AMB2016/AMB2018 visual cortex populations attach to the correct parents.

## When NOT to use

- Datasets are not yet aligned — scHPL is not a batch correction tool, so running tree construction/comparison on unaligned data makes it impossible to tell whether tree mistakes come from erroneous alignment or from scHPL mismatches
- The cell populations of interest are absent from both the learned tree and any published reference (e.g. many brain cell populations are missing from the Cell Ontology), leaving nothing to compare against

## Inputs

- Seurat-aligned AMB2016 AnnData object (integrated matrix in .X, batch and population labels in metadata)
- Seurat-aligned AMB2018 AnnData object (same structure)
- Published reference hierarchy (Tasic et al. Extended Data Fig. 6 cluster correspondences)

## Outputs

- Learned hierarchical classification tree structure (exported)
- Edge-level comparison report of learned relationships vs. reference hierarchy
- Per-node trained linear SVM classifiers

## How to apply

After constructing the tree with scHPL.learn on the Seurat-aligned AMB2016 and AMB2018 AnnData objects (integrated expression matrix in .X, batch indicators and population labels in metadata), export the learned tree structure and inspect its parent-child edges. Compare each learned relationship against the reference hierarchy from Tasic et al. Extended Data Fig. 6, focusing on edge-level correspondences rather than global shape alone. Specific checks include: 'L6b VISp Col8a1 Rprm' should attach as a subpopulation of 'L6b Sla' rather than 'L6b Rgs12', and the three L4 populations should merge into a single branch. Train a linear SVM at each node, since integrated data retaining many dimensions is better served by linear SVMs (accuracy equal to or higher than the one-class SVM, especially on small populations). Near-perfect agreement with the published hierarchy, including these two specific structural features, indicates correct reconstruction.

## Related tools

- **scHPL** (Constructs the hierarchical classification tree via scHPL.learn on the two aligned datasets and trains per-node classifiers (linear SVM) whose tree is compared to the published hierarchy) — https://github.com/lcmmichielsen/scHPL
- **Seurat** (Used to align the AMB2016 and AMB2018 datasets before scHPL matches cell populations, producing the integrated .X matrix fed into tree learning)
- **scikit-learn** (Provides the linear SVM trained at each node of the learned tree)

## Examples

```
from scHPL import learn; tree = learn(amb2016_adata, amb2018_adata, classifier='linear SVM')
```

## Evaluation signals

- 'L6b VISp Col8a1 Rprm' is placed as a subpopulation of 'L6b Sla', not under 'L6b Rgs12'
- The three L4 visual cortex populations merge into a single branch in the learned tree
- Overall learned tree shows near-perfect agreement with the cluster correspondences in Tasic et al. Extended Data Fig. 6
- A linear SVM is trained at every tree node (advised for high-dimensional integrated data), with no population left without a classifier
- Exported tree structure can be re-inspected and every parent-child edge traced to a correspondence in the reference figure

## Limitations

- With current batch correction tools, adding a new dataset requires rerunning the alignment and consequently scHPL, invalidating the previously validated tree
- When datasets are imperfectly aligned, it is hard to resolve whether mistakes in the constructed tree are caused by erroneous alignment or by mismatches created by scHPL
- The hierarchical-relationship assumption itself can fail (e.g. dedifferentiation such as beta-to-alpha cell conversions in type 2 diabetes), so a tree comparison may be inappropriate for such biology

## Evidence

- [results] almost perfect learned tree: [evidence span withheld — anchor status: not_found]
- [discussion] Col8a1 Rprm mapping: "such as the mapping of “L6b VISp Col8a1 Rprm” as a subpopulation of “L6b Sla” instead of “L6b Rgs12”"
- [discussion] alignment requirement: "Since batch effects are inevitable when combining datasets, we require datasets to be aligned before running scHPL"
- [methods] tree construction: "scHPL is a method that automatically finds relationships between cell populations across multiple datasets and uses this to construct a hierarchical classification tree."
- [readme] reproducibility notebooks: "The `vignette folder </vignettes>`_ contains notebooks to reproduce the inter-dataset experiments."
