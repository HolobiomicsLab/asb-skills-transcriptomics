---
name: cross-dataset-cell-annotation-evaluation
description: Use when you have 2+ annotated scRNA-seq reference datasets with batch
  effects and a held-out unlabeled target dataset, and you need cross-dataset cell-type
  annotation plus an evaluation against known labels.
when_to_use_negative:
- 'Datasets are not yet aligned: scHPL is explicitly not a batch correction tool and
  advises alignment (e.g., with Seurat v3, scVI, or scArches) before matching cell
  populations'
- The analysis requires adding a new dataset to an existing tree without rerunning
  integration — with current batch correction tools the alignment and scHPL must be
  rerun
- The biological system violates hierarchical cell-type relationships, e.g., dedifferentiation
  states like beta-to-alpha cell conversions in type 2 diabetes
edam_operation: http://edamontology.org/operation_1777
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scHPL
  role: 'Core method: learn the hierarchical classification tree from aligned annotated
    datasets, train per-node linear SVM, predict labels on the query dataset, and
    evaluate with the evaluate module'
  repo: https://github.com/lcmmichielsen/scHPL
- name: Seurat
  role: Integrates the reference PBMC datasets (2000 highly variable genes) to remove
    batch effects before scHPL population matching; linear SVM is the advised classifier
    for Seurat-integrated data
- name: scArches
  role: Alternative alignment framework recommended by the README (via the treeArches
    framework) for integrating datasets before running scHPL
  repo: https://github.com/theislab/scarches
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-23196-8
    title: Hierarchical progressive learning of cell identities in single-cell data
  source_content_hash: sha256:234222fe6ce15cc3d164804023a65d9baa50877786c80cbf15d63b7221ada3aa
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
yes_rate: 0.4444444444444444
n_claims: 9
---

# cross-dataset-cell-annotation-evaluation

## Summary

Train a hierarchical classification tree with scHPL on multiple annotated reference datasets and evaluate it by predicting labels on an independent held-out dataset, using Seurat v3 alignment and a linear SVM classifier. This skill reproduces the PBMC inter-dataset experiment (Fig. 5) where a tree learned from three PBMC datasets annotates PBMC-Bench10Xv3.

## When to use

Use when you have 2+ annotated scRNA-seq reference datasets with batch effects and a held-out unlabeled target dataset, and you need cross-dataset cell-type annotation plus an evaluation against known labels.

## When NOT to use

- Datasets are not yet aligned: scHPL is explicitly not a batch correction tool and advises alignment (e.g., with Seurat v3, scVI, or scArches) before matching cell populations
- The analysis requires adding a new dataset to an existing tree without rerunning integration — with current batch correction tools the alignment and scHPL must be rerun
- The biological system violates hierarchical cell-type relationships, e.g., dedifferentiation states like beta-to-alpha cell conversions in type 2 diabetes

## Inputs

- Three or more annotated single-cell reference datasets (e.g., PBMC-eQTL, PBMC-Bench10Xv2, PBMC-FACS) as AnnData objects with cell-type labels and batch indicators
- Seurat v3-integrated expression matrix (2000 highly variable genes) placed in the '.X' slot of an AnnData object
- One held-out unlabeled query dataset for prediction (e.g., PBMC-Bench10Xv3, downloadable from the scHPL Zenodo record doi:10.5281/zenodo.4557712)

## Outputs

- Learned hierarchical classification tree matching cell populations across datasets (Fig. 5B-style tree figure)
- Predicted cell-type labels for every cell of the held-out dataset
- Hierarchy-aware confusion matrix comparing predicted vs. known labels (Fig. 5C-style)
- Identified mislabeled or mismatched populations, diagnosed via marker gene expression

## How to apply

First align the annotated reference datasets before running scHPL, since scHPL is not a batch correction tool; in the PBMC workflow this was done by integrating PBMC-eQTL, PBMC-Bench10Xv2, and PBMC-FACS with Seurat v3 using 2000 highly variable genes, then copying the integrated expression matrix into the '.X' slot of an AnnData object with labels and batch indicators as metadata. Run scHPL's learn function on the aligned datasets to automatically match cell populations across datasets and construct the hierarchical classification tree. Train the advised classifier at each tree node — a linear SVM for Seurat-integrated data, which retains many dimensions — and then apply scHPL's predict function to label cells of the held-out dataset (e.g., PBMC-Bench10Xv3). Finally, use scHPL's evaluate module to produce the learned tree figure and a hierarchy-aware confusion matrix, and interpret errors against marker gene expression, since observed misclassifications (e.g., CD16+ monocytes vs mDC) can stem from mislabeling in the training data rather than classifier failure.

## Related tools

- **scHPL** (Core method: learn the hierarchical classification tree from aligned annotated datasets, train per-node linear SVM, predict labels on the query dataset, and evaluate with the evaluate module) — https://github.com/lcmmichielsen/scHPL
- **Seurat** (Integrates the reference PBMC datasets (2000 highly variable genes) to remove batch effects before scHPL population matching; linear SVM is the advised classifier for Seurat-integrated data)
- **scArches** (Alternative alignment framework recommended by the README (via the treeArches framework) for integrating datasets before running scHPL) — https://github.com/theislab/scarches

## Examples

```
import scHPL; tree = scHPL.learn(adata_train, classifier='linear SVM'); predictions = scHPL.predict(tree, adata_query)
```

## Evaluation signals

- The learned tree and confusion matrix match the published PBMC inter-dataset results (Figs. 5B and 5C), verified by output_matches_reference and expert_review of the tree structure
- Most cells of the held-out PBMC-Bench10Xv3 dataset are correctly annotated; specific confusions (e.g., CD16+ monocytes vs mDC) are explainable by mislabeling in the training datasets, confirmed by marker gene expression
- Original annotations from each reference dataset are preserved while building the tree (scHPL learns hierarchies 'while preserving the original annotations')
- Linear SVM per-node performance is at or above one-class SVM performance, particularly on small cell populations where one-class SVM degrades more
- scHPL is run only after a verified alignment step; running it on unaligned data is an invalid invocation of this skill

## Limitations

- scHPL is not a batch correction tool; datasets must be aligned first, and when a new dataset is added the alignment—and consequently scHPL—must be rerun
- Batch effects make it hard to determine whether tree mistakes stem from erroneous alignment or mismatches created by scHPL
- Training datasets can contain mislabeled populations (detected here via marker gene expression), which propagate as apparent classifier errors
- The hierarchical-relationship assumption has inherent limits, e.g., dedifferentiation such as beta-to-alpha cell conversions in type 2 diabetes
- A constant terminology for cell populations across datasets is missing, complicating consistent annotation

## Evidence

- [discussion] Datasets must be aligned before running scHPL: "Since batch effects are inevitable when combining datasets, we require datasets to be aligned before running scHPL"
- [methods] Per-node classifiers: "For each node in the tree either a linear SVM, kNN, or one-class SVM is trained."
- [methods] Prediction workflow: "The trained classification tree can be used to predict the labels of a new unlabeled dataset."
- [discussion] Linear vs one-class SVM: "For both classifiers, we saw a decrease in performance on populations with a small number of cells, but for the one-class SVM this effect was more apparent."
- [discussion] Mislabeled populations detected: "the expression of marker genes revealed that (parts of) several cell populations were mislabeled"
- [readme] Vignettes reproduce experiments: "The `vignette folder </vignettes>`_ contains notebooks to reproduce the inter-dataset experiments."
