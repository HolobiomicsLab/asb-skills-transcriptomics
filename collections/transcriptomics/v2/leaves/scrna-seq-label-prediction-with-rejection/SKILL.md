---
name: scrna-seq-label-prediction-with-rejection
description: Use when annotating scRNA-seq datasets from different labs or protocols
  using labeled reference datasets, especially when annotations are at different resolutions
  and unknown cell populations must be rejected or discovered.
when_to_use_negative:
- Datasets are not batch-corrected/aligned — scHPL explicitly requires alignment beforehand
  and does not perform batch correction itself
- Cell populations do not follow a hierarchical (tree-like) relationship, e.g., dedifferentiation
  states such as beta-to-alpha cell conversions in type 2 diabetes
- A complete reference atlas with standardized nomenclature already resolves the annotation
  task — simpler name-based or marker-based annotation may suffice
edam_operation: http://edamontology.org/operation_1777
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scHPL
  role: 'Core method: learns population relationships, builds the tree, trains per-node
    classifiers, predicts labels with rejection, and evaluates'
  repo: https://github.com/lcmmichielsen/scHPL
- name: Seurat
  role: Aligns datasets (batch correction) before scHPL matching, with aligned expression
    placed in AnnData .X
- name: scArches
  role: Alternative recommended alignment tool within the treeArches framework for
    building and updating reference atlases
  repo: https://github.com/theislab/scarches
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-23196-8
    title: Hierarchical progressive learning of cell identities in single-cell data
  source_content_hash: sha256:a04ca946b822b9855f9ba234acd16c6f589bbe13bbd827f9b94cea209c728800
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lcmmichielsen/scHPL
  tools:
  - Seurat
  - scArches
  - scHPL
  edam_operation: http://edamontology.org/operation_1777
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# scRNA-seq label prediction with rejection

## Summary

Use scHPL, a hierarchical progressive learning method, to annotate cells in single-cell RNA-seq data by constructing a classification tree across multiple datasets and training per-node classifiers (linear SVM, kNN, or one-class SVM) so that unseen populations can be rejected and new cell populations discovered.

## When to use

Use when annotating scRNA-seq datasets from different labs or protocols using labeled reference datasets, especially when annotations are at different resolutions and unknown cell populations must be rejected or discovered.

## When NOT to use

- Datasets are not batch-corrected/aligned — scHPL explicitly requires alignment beforehand and does not perform batch correction itself
- Cell populations do not follow a hierarchical (tree-like) relationship, e.g., dedifferentiation states such as beta-to-alpha cell conversions in type 2 diabetes
- A complete reference atlas with standardized nomenclature already resolves the annotation task — simpler name-based or marker-based annotation may suffice

## Inputs

- Aligned scRNA-seq AnnData objects with expression matrices in the .X slot and cell label / batch indicator metadata columns (e.g., Seurat-aligned Saunders, Zeisel, Tabula Muris datasets)
- An unlabeled query scRNA-seq dataset for prediction (e.g., Seurat-aligned Rosenberg dataset)
- Reference datasets downloaded from the scHPL Zenodo archive (https://doi.org/10.5281/zenodo.4557712)

## Outputs

- A hierarchical classification tree of cell populations linking populations across datasets
- Per-node trained classifiers (linear SVM, kNN, or one-class SVM)
- Per-cell predicted labels with rejection assignments for the query dataset
- Evaluation metrics of classifier accuracy per node/dataset

## How to apply

First align the datasets (e.g., with Seurat, scVI, or scArches), since scHPL is not a batch correction tool and requires aligned inputs before matching cell populations. Run scHPL.learn starting from an annotated dataset and progressively add other datasets so scHPL automatically finds relationships between cell populations and constructs the hierarchical tree. Train a classifier at each tree node with scHPL.train — choose the linear SVM for integrated data with many dimensions (e.g., Seurat-integrated data), or one-class SVM when rejection of unknown populations is the priority. Apply scHPL.predict to a new unlabeled dataset; cells that match no tree node are rejected, and rejected populations can be inspected and added to the tree. Finally, evaluate per-node and overall performance with scHPL.evaluate, comparing linear SVM against the one-class SVM baseline.

## Related tools

- **scHPL** (Core method: learns population relationships, builds the tree, trains per-node classifiers, predicts labels with rejection, and evaluates) — https://github.com/lcmmichielsen/scHPL
- **Seurat** (Aligns datasets (batch correction) before scHPL matching, with aligned expression placed in AnnData .X)
- **scArches** (Alternative recommended alignment tool within the treeArches framework for building and updating reference atlases) — https://github.com/theislab/scarches

## Examples

```
pip install scHPL; tree = scHPL.learn(adata_saunders); tree = scHPL.learn.add_labels(tree, adata_zeisel); scHPL.train(tree, classifier='linear'); predictions = scHPL.predict(tree, adata_rosenberg)
```

## Evaluation signals

- The learned tree reproduces reported inter-dataset matches, e.g., Saunders mural cells matched to Tabula Muris brain pericytes, consistent with Fig. 7G
- Predictions on the independent query dataset match the published assignments (e.g., Rosenberg dataset annotations in Fig. 7G), with sensible rejection of non-matching cells
- Per-node linear SVM accuracy is equal to or higher than the one-class SVM baseline; one-class SVM performance drops more on small populations
- The tree structure matches the known hierarchy in the source paper (e.g., L6b subpopulation relationships reported in the discussion)
- Erroneous tree branches can be checked via marker gene expression, as done to detect mislabeled PBMC populations

## Limitations

- scHPL is not a batch correction tool — datasets must be aligned first, and adding a new dataset requires rerunning the alignment and scHPL
- Batch effects make it hard to determine whether tree mistakes stem from erroneous alignment or mismatches created by scHPL
- One-class SVM performance decreases more than linear SVM on populations with a small number of cells
- The tree assumption fails when cell identities are not hierarchical, e.g., dedifferentiation conversions like beta-to-alpha cells in type 2 diabetes
- A constant terminology for cell populations across datasets is missing, and many brain cell populations are absent from the Cell Ontology, so validation of matches can be limited

## Evidence

- [readme] hierarchical progressive learning method which automatically finds relationships between cell populations across multiple datasets: "hierarchical progressive learning method which automatically finds relationships between cell populations across multiple datasets and uses this to construct a hierarchical classification tree"
- [methods] For each node in the tree either a linear SVM, kNN, or one-class SVM is trained: "For each node in the tree either a linear SVM, kNN, or one-class SVM is trained."
- [readme] we advise to align the datasets before matching the cell populations: "NOTE: scHPL is not a batch correction tool, we advise to align the datasets before matching the cell populations."
- [abstract] scHPL can successfully learn known cellular hierarchies from multiple datasets while preserving the original annotations: "show that scHPL can successfully learn known cellular hierarchies from multiple datasets while preserving the original annotations"
- [discussion] for the one-class SVM this effect was more apparent: "For both classifiers, we saw a decrease in performance on populations with a small number of cells, but for the one-class SVM this effect was more apparent."
