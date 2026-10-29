---
name: hierarchy-aware-confusion-matrix-analysis
description: Use when a classification tree (e.g. scHPL trained on multiple annotated
  scRNA-seq datasets) has predicted labels for a held-out dataset and you need to
  quantify which cross-node and within-subtree misclassifications occur, as in the
  PBMC Bench10Xv3 evaluation.
when_to_use_negative:
- scHPL has not yet been applied — this skill evaluates tree predictions and requires
  an already-trained classification tree and predicted labels
- Datasets are not batch-corrected/aligned — scHPL requires datasets to be aligned
  (e.g. via Seurat v3, scVI, or scArches) before learning and prediction, so evaluation
  would be confounded by batch effects
- The task is a flat (non-hierarchical) cell-type classification where per-population
  confusions have no tree structure to exploit
tools:
- name: scHPL
  role: Provides the predict function to label query cells and the evaluate module
    that generates the learned tree figure and the hierarchy-aware confusion matrix
  repo: https://github.com/lcmmichielsen/scHPL
- name: Seurat
  role: Used upstream to integrate PBMC datasets (v3 integration, 2000 highly variable
    genes) so the tree predictions evaluated here are batch-corrected
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-23196-8
    title: Hierarchical progressive learning of cell identities in single-cell data
  source_content_hash: sha256:5fc289ba2787e2382ed490ce1e0db64e932659449193c9786e4276685f8dc8a4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lcmmichielsen/scHPL
  tools:
  - Seurat
  - scHPL
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# Hierarchy-aware confusion matrix analysis

## Summary

Evaluate cell-identity predictions from a hierarchical classifier (scHPL) by constructing a confusion matrix whose misclassification counts respect the tree's hierarchy boundaries, rather than treating all cell-population confusions as equally severe errors.

## When to use

Use when a classification tree (e.g. scHPL trained on multiple annotated scRNA-seq datasets) has predicted labels for a held-out dataset and you need to quantify which cross-node and within-subtree misclassifications occur, as in the PBMC Bench10Xv3 evaluation.

## When NOT to use

- scHPL has not yet been applied — this skill evaluates tree predictions and requires an already-trained classification tree and predicted labels
- Datasets are not batch-corrected/aligned — scHPL requires datasets to be aligned (e.g. via Seurat v3, scVI, or scArches) before learning and prediction, so evaluation would be confounded by batch effects
- The task is a flat (non-hierarchical) cell-type classification where per-population confusions have no tree structure to exploit

## Inputs

- Predicted cell labels for a held-out dataset (e.g. PBMC-Bench10Xv3) from a trained scHPL classification tree
- Ground-truth cell labels of the query dataset
- The learned hierarchical classification tree (population relationships across PBMC-eQTL, PBMC-Bench10Xv2, PBMC-FACS)

## Outputs

- Confusion matrix whose structure reflects the tree hierarchy boundaries (Fig. 5C analogue)
- Learned tree figure (Fig. 5B analogue)
- Identified systematically misclassified populations with suspected training-label errors (e.g. CD16+ monocytes vs mDC)

## How to apply

After training the scHPL tree with a linear SVM per node (the advised classifier for Seurat v3-integrated data retaining many dimensions), apply scHPL's predict function to the unlabeled query dataset (e.g. PBMC-Bench10Xv3 cells). Then run scHPL's evaluate module to generate the learned tree figure (Fig. 5B) and a confusion matrix that reflects the hierarchy boundaries (Fig. 5C), so errors between sibling populations are distinguished from errors across distant branches. Inspect systematic off-diagonal entries (e.g. CD16+ monocytes vs mDC) as candidate label errors in the training data rather than classifier failure, verifying suspected mislabeling via marker gene expression. Validate by comparing the tree and confusion matrix against the published PBMC inter-dataset results in Figs. 5B and 5C.

## Related tools

- **scHPL** (Provides the predict function to label query cells and the evaluate module that generates the learned tree figure and the hierarchy-aware confusion matrix) — https://github.com/lcmmichielsen/scHPL
- **Seurat** (Used upstream to integrate PBMC datasets (v3 integration, 2000 highly variable genes) so the tree predictions evaluated here are batch-corrected)

## Examples

```
import scHPL; pred = scHPL.predict(X_query, tree, classifier='svm'); scHPL.evaluate(pred, y_true, tree)
```

## Evaluation signals

- The generated tree figure and confusion matrix match the published PBMC inter-dataset results (Figs. 5B and 5C) via output_matches_reference
- Expert review confirms the learned tree structure matches the reference hierarchy
- Most Bench10Xv3 cells are correctly annotated; remaining off-diagonal counts are localized (e.g. CD16+ monocytes vs mDC) rather than diffuse
- Suspected systematic confusions are corroborated by marker gene expression indicating mislabeling in the training datasets
- Misclassification rates behave as the paper reports: linear SVM accuracy is equal to or higher than one-class SVM, with one-class SVM degrading more on small populations

## Limitations

- Observed misclassifications can reflect mislabeled populations in the training datasets rather than classifier errors
- scHPL is not a batch correction tool; with batch effects it is hard to determine whether confusion stems from erroneous alignment or from mismatches created by scHPL
- Performance decreases on populations with a small number of cells

## Evidence

- [methods] The trained classification tree can be used to predict the labels of a new unlabeled dataset.: "The trained classification tree can be used to predict the labels of a new unlabeled dataset."
- [readme] For each node in the tree either a linear SVM, kNN, or one-class SVM, which enables the detection of unknown populations, is trained.: "For each node in the tree either a linear SVM, kNN, or one-class SVM, which enables the detection of unknown populations, is trained."
- [discussion] the expression of marker genes revealed that (parts of) several cell populations were mislabeled: "the expression of marker genes revealed that (parts of) several cell populations were mislabeled"
- [readme] The vignette folder contains notebooks to reproduce the inter-dataset experiments.: "The vignette folder </vignettes>`_ contains notebooks to reproduce the inter-dataset experiments"
