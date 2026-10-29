---
name: cross-dataset-cell-population-matching
description: Use when two or more annotated scRNA-seq datasets (batch-corrected/aligned,
  e.g. with scVI or Seurat) must have their cell population labels matched and a hierarchical
  classification tree learned, especially when name-based matching fails (e.g. different
  labs, resolutions, or nomenclatures).
when_to_use_negative:
- Datasets are not yet batch-corrected/aligned — scHPL is explicitly not a batch correction
  tool and requires alignment (scVI, scArches, Seurat) beforehand
- Cell populations violate hierarchical/tree relationships, e.g. dedifferentiation
  cases such as beta-to-alpha cell conversions in type 2 diabetes
- A new dataset is added but rerunning the full alignment is not feasible — with current
  batch correction tools the alignment and consequently scHPL must be rerun
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: scHPL
  role: 'Core method: learns relationships between cell populations across datasets
    via scHPL.learn, builds and updates the hierarchical classification tree, trains
    per-node classifiers (scHPL.train), predicts (scHPL.predict) and evaluates (scHPL.evaluate)'
  repo: https://github.com/lcmmichielsen/scHPL
- name: scArches
  role: Recommended batch correction/alignment of datasets before scHPL matches cell
    populations (via the treeArches framework)
  repo: https://github.com/theislab/scarches
- name: Seurat
  role: Alignment/integration of the AMB2016, AMB2018, Saunders, Zeisel, Tabula Muris,
    and Rosenberg datasets before running scHPL
- name: scikit-learn
  role: Provides the linear SVM trained at each node of the learned tree for integrated
    data retaining many dimensions
provenance:
  source_task_ids:
  - task_005
  - task_006
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-23196-8
    title: Hierarchical progressive learning of cell identities in single-cell data
  source_content_hash: sha256:dd433794562ef50d053c458627dfca117ecb6c429278855ab4b0d30429a041b6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lcmmichielsen/scHPL
  tools:
  - Seurat
  - scArches
  - scHPL
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# cross-dataset-cell-population-matching

## Summary

A skill for matching cell population labels across scRNA-seq datasets using scHPL's hierarchical progressive learning: cross-predictions between datasets yield confusion matrices from which a matching matrix is built, driving tree updates (match, merge, split, new population) and construction of a hierarchical classification tree.

## When to use

Use when two or more annotated scRNA-seq datasets (batch-corrected/aligned, e.g. with scVI or Seurat) must have their cell population labels matched and a hierarchical classification tree learned, especially when name-based matching fails (e.g. different labs, resolutions, or nomenclatures).

## When NOT to use

- Datasets are not yet batch-corrected/aligned — scHPL is explicitly not a batch correction tool and requires alignment (scVI, scArches, Seurat) beforehand
- Cell populations violate hierarchical/tree relationships, e.g. dedifferentiation cases such as beta-to-alpha cell conversions in type 2 diabetes
- A new dataset is added but rerunning the full alignment is not feasible — with current batch correction tools the alignment and consequently scHPL must be rerun

## Inputs

- Aligned, labeled AnnData objects (integrated expression matrix in .X, cell labels and batch indicators in metadata) for each dataset, e.g. Seurat-aligned or scVI/scArches-integrated
- Datasets annotated at different resolutions (e.g. low-resolution Zeisel annotations vs high-resolution Tabula Muris or AMB2016/AMB2018 visual cortex data)
- An independent unlabeled query dataset for prediction (e.g. Seurat-aligned Rosenberg dataset)

## Outputs

- Hierarchical classification tree with cross-dataset population matches, merges, splits, and new-population nodes (e.g. learned_tree.pkl)
- Matching matrix X summarizing population correspondences between datasets
- Per-cell predicted labels with rejection option for a query dataset, plus per-node classifier evaluation

## How to apply

Align datasets first — scHPL is not a batch correction tool, so integrate with scVI, scArches, or Seurat, placing the integrated matrix in the AnnData .X slot and labels/batch indicators in metadata. Train classifiers on dataset D1's tree and predict D2 to obtain cross-prediction confusion matrices; normalize to per-population fractions and binarize (BC1, BC2) using a fraction-ranking threshold of 0.25 to decide matched population pairs. Compute the matching matrix X as the sum of BC1's transpose and BC2 to identify one-to-one matches, merges, splits, and unmatched populations, then update the tree accordingly: attach unmatched D2 populations as new nodes, merge split populations under a shared parent, and relabel matched nodes. Train a classifier at each node (linear SVM is advised for integrated data retaining many dimensions, e.g. Seurat-aligned data; kNN or one-class SVM alternatives exist, with the one-class SVM enabling rejection of unknown populations). Persist the tree (e.g. learned_tree.pkl) and optionally apply scHPL.predict to an independent dataset followed by scHPL.evaluate.

## Related tools

- **scHPL** (Core method: learns relationships between cell populations across datasets via scHPL.learn, builds and updates the hierarchical classification tree, trains per-node classifiers (scHPL.train), predicts (scHPL.predict) and evaluates (scHPL.evaluate)) — https://github.com/lcmmichielsen/scHPL
- **scArches** (Recommended batch correction/alignment of datasets before scHPL matches cell populations (via the treeArches framework)) — https://github.com/theislab/scarches
- **Seurat** (Alignment/integration of the AMB2016, AMB2018, Saunders, Zeisel, Tabula Muris, and Rosenberg datasets before running scHPL)
- **scikit-learn** (Provides the linear SVM trained at each node of the learned tree for integrated data retaining many dimensions)

## Examples

```
import scHPL; tree = scHPL.learn(adata1, adata2); scHPL.train(tree, classifier='linear_svm'); labels = scHPL.predict(tree, adata_query)
```

## Evaluation signals

- Learned tree reproduces known cross-dataset correspondences, e.g. Saunders mural cells matching Tabula Muris brain pericytes (Fig. 7G)
- For AMB2016/AMB2018: 'L6b VISp Col8a1 Rprm' attaches under 'L6b Sla' (not 'L6b Rgs12') and the three L4 populations merge, matching Tasic et al. Extended Data Fig. 6
- Every D2 population in the reloaded learned_tree.pkl is either matched to a D1 node or attached as a new node
- Per-node linear SVM accuracy is equal to or higher than the one-class SVM baseline on all datasets
- Independent query dataset (e.g. Rosenberg) predictions with rejection match the published assignments

## Limitations

- Batch effects make it hard to determine whether tree mistakes stem from erroneous alignment or mismatches created by scHPL
- Adding a new dataset requires rerunning the alignment and consequently scHPL with current batch correction tools
- The hierarchical-relationship assumption can fail, e.g. dedifferentiation like beta-to-alpha cell conversions in type 2 diabetes
- Performance decreases on populations with a small number of cells, more pronounced for the one-class SVM
- Many brain cell populations are missing from the Cell Ontology and annotation consistency between studies is lacking, so no external ontology check is available

## Evidence

- [methods] matching matrix construction: "scHPL is a method that automatically finds relationships between cell populations across multiple datasets and uses this to construct a hierarchical classification tree."
- [methods] classifier options: "For each node in the tree either a linear SVM, kNN, or one-class SVM is trained."
- [discussion] alignment prerequisite: "Since batch effects are inevitable when combining datasets, we require datasets to be aligned before running scHPL"
- [readme] README alignment note: "NOTE: scHPL is not a batch correction tool, we advise to align the datasets before matching the cell populations. We advise doing this with scVI or scArches"
- [results] cross-dataset match example: [evidence span withheld — anchor status: not_found]
