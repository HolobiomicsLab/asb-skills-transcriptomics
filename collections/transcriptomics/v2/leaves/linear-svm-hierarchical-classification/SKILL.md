---
name: linear-svm-hierarchical-classification
description: Use when you have multiple aligned (batch-corrected), annotated scRNA-seq
  datasets with high-dimensional integrated expression matrices and need to construct
  or extend a hierarchical cell-population classification tree with per-node linear
  SVMs.
when_to_use_negative:
- Input datasets are not aligned — scHPL is explicitly not a batch correction tool,
  so unaligned datasets must first be corrected (e.g., with Seurat, scVI, or scArches)
- You only need to annotate an unlabeled dataset against an existing reference without
  discovering relationships between populations
- Cell populations are expected to violate strict hierarchical relationships (e.g.,
  dedifferentiation such as beta-to-alpha cell conversions in type 2 diabetes)
edam_operation: http://edamontology.org/operation_2995
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: scHPL
  role: Constructs the hierarchical classification tree by matching cell populations
    across datasets and trains a classifier (linear SVM, kNN, or one-class SVM) at
    each node
  repo: https://github.com/lcmmichielsen/scHPL
- name: Seurat
  role: Used to align the AMB2016 and AMB2018 visual cortex datasets before scHPL
    population matching
- name: scikit-learn
  role: Provides the linear SVM trained at each node of the learned tree
- name: scArches
  role: Recommended by the README for aligning datasets before scHPL matches cell
    populations
  repo: https://github.com/theislab/scarches
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-23196-8
    title: Hierarchical progressive learning of cell identities in single-cell data
  source_content_hash: sha256:d0a7bc1999e2d9a9d0fe04b6ade2dc55774d31c1d9d4dd1fd04b6ec9f19ddf70
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lcmmichielsen/scHPL
  tools:
  - Seurat
  - scArches
  - scHPL
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2995
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# linear-svm-hierarchical-classification

## Summary

Train a hierarchical classification tree with a linear SVM at each node using scHPL, so that cell populations annotated at different resolutions (e.g., Seurat-aligned AMB2016 vs AMB2018 mouse visual cortex) are automatically related and new populations can be attached at the correct tree position. The linear SVM is the advised classifier for integrated data that retains many dimensions.

## When to use

Use when you have multiple aligned (batch-corrected), annotated scRNA-seq datasets with high-dimensional integrated expression matrices and need to construct or extend a hierarchical cell-population classification tree with per-node linear SVMs.

## When NOT to use

- Input datasets are not aligned — scHPL is explicitly not a batch correction tool, so unaligned datasets must first be corrected (e.g., with Seurat, scVI, or scArches)
- You only need to annotate an unlabeled dataset against an existing reference without discovering relationships between populations
- Cell populations are expected to violate strict hierarchical relationships (e.g., dedifferentiation such as beta-to-alpha cell conversions in type 2 diabetes)

## Inputs

- Two or more annotated scRNA-seq AnnData objects with aligned/integrated expression matrices in .X and batch indicators and population labels in metadata (e.g., Seurat-aligned AMB2016 and AMB2018 mouse primary visual cortex datasets)

## Outputs

- A hierarchical classification tree relating cell populations across datasets
- A trained linear SVM classifier at each node of the tree
- Exported tree structure for comparison against a reference hierarchy (e.g., Tasic et al. Extended Data Fig. 6)

## How to apply

First align the datasets (e.g., with Seurat, scVI, or scArches), since scHPL is not a batch-correction tool and requires aligned inputs; place the integrated expression matrix in AnnData .X and batch/label annotations in the metadata. Call scHPL.learn on the aligned datasets so it automatically matches cell populations across datasets and builds the hierarchical classification tree. For integrated data retaining many dimensions, train a linear SVM at each node rather than a one-class SVM, as the linear SVM showed accuracy equal to or higher than the one-class SVM on all datasets and degrades less on small populations. Inspect and export the learned tree, then compare its structure against the known reference hierarchy — e.g., verifying that 'L6b VISp Col8a1 Rprm' attaches under 'L6b Sla' rather than 'L6b Rgs12' and that the three L4 populations merge.

## Related tools

- **scHPL** (Constructs the hierarchical classification tree by matching cell populations across datasets and trains a classifier (linear SVM, kNN, or one-class SVM) at each node) — https://github.com/lcmmichielsen/scHPL
- **Seurat** (Used to align the AMB2016 and AMB2018 visual cortex datasets before scHPL population matching)
- **scikit-learn** (Provides the linear SVM trained at each node of the learned tree)
- **scArches** (Recommended by the README for aligning datasets before scHPL matches cell populations) — https://github.com/theislab/scarches

## Examples

```
pip install scHPL
```

## Evaluation signals

- The learned tree matches the published reference hierarchy (Tasic et al. Extended Data Fig. 6) with near-perfect agreement for the AMB2016–AMB2018 comparison
- 'L6b VISp Col8a1 Rprm' is assigned as a subpopulation of 'L6b Sla' rather than 'L6b Rgs12'
- The three L4 populations merge into a single branch of the learned tree
- Linear SVM classifier accuracy at each node is equal to or higher than a one-class SVM baseline, especially on small populations
- Datasets were confirmed aligned before running scHPL, so tree mistakes can be attributed to scHPL matching rather than erroneous alignment

## Limitations

- Datasets must be aligned before running scHPL; adding a new dataset requires rerunning the alignment and consequently scHPL
- When batch effects are present, it is hard to resolve whether mistakes in the constructed tree come from erroneous alignment or from mismatches created by scHPL
- For small populations, classifier performance decreases — the effect is more apparent for the one-class SVM but affects both classifiers
- The hierarchical-relationship assumption is inherent; situations such as dedifferentiation (e.g., beta-to-alpha cell conversions in type 2 diabetes) are not adequately represented by a tree
- Many brain cell populations are missing from the Cell Ontology, limiting external verification of learned relationships

## Evidence

- [methods] Per-node linear SVM training: "For each node in the tree either a linear SVM, kNN, or one-class SVM is trained."
- [discussion] Alignment requirement before scHPL: "Since batch effects are inevitable when combining datasets, we require datasets to be aligned before running scHPL"
- [discussion] Linear SVM vs one-class SVM performance on small populations: "For both classifiers, we saw a decrease in performance on populations with a small number of cells, but for the one-class SVM this effect was more apparent."
- [readme] scHPL is not a batch correction tool (README): "scHPL is not a batch correction tool, we advise to align the datasets before matching the cell populations."
- [methods] scHPL automatically constructs hierarchical classification tree: "scHPL is a method that automatically finds relationships between cell populations across multiple datasets and uses this to construct a hierarchical classification tree."
