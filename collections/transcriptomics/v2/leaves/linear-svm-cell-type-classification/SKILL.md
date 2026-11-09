---
name: linear-svm-cell-type-classification
description: Use when annotating cells of an aligned, Seurat-v3-integrated scRNA-seq
  dataset whose integrated expression matrix (retaining many dimensions) is placed
  in the .X slot of an AnnData object, and a hierarchical classifier is trained per
  tree node in scHPL.
when_to_use_negative:
- The input datasets have not been aligned first — scHPL is not a batch correction
  tool, so datasets must be aligned (e.g., with Seurat, scVI, or scArches) before
  matching cell populations
- The goal is batch correction or atlas construction/updating itself rather than hierarchical
  cell-type classification
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scHPL
  role: Learns the hierarchical classification tree and trains the linear SVM at each
    node; its predict and evaluate functions label query cells and produce the tree
    figure and confusion matrix
  repo: https://github.com/lcmmichielsen/scHPL
- name: Seurat
  role: Used to batch-align the training datasets (v3 integration on 2000 highly variable
    genes), producing the high-dimensional integrated matrix for which the linear
    SVM is the advised node classifier
- name: scArches
  role: Advised alternative alignment tool before running scHPL, within the treeArches
    framework
  repo: https://github.com/theislab/scarches
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-23196-8
    title: Hierarchical progressive learning of cell identities in single-cell data
  source_content_hash: sha256:d53fa3afd23ec19338a370f68552bd7894c4fad01016072bf5efae0c3c852f25
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lcmmichielsen/scHPL
  tools:
  - Seurat
  - scArches
  - scHPL
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# linear-svm-cell-type-classification

## Summary

Train a linear SVM classifier at each node of an scHPL hierarchical classification tree, the advised classifier for Seurat-integrated single-cell data that retains many dimensions, and use the trained tree to predict cell-type labels of a held-out dataset.

## When to use

Use when annotating cells of an aligned, Seurat-v3-integrated scRNA-seq dataset whose integrated expression matrix (retaining many dimensions) is placed in the .X slot of an AnnData object, and a hierarchical classifier is trained per tree node in scHPL.

## When NOT to use

- The input datasets have not been aligned first — scHPL is not a batch correction tool, so datasets must be aligned (e.g., with Seurat, scVI, or scArches) before matching cell populations
- The goal is batch correction or atlas construction/updating itself rather than hierarchical cell-type classification

## Inputs

- Multiple annotated, batch-aligned scRNA-seq datasets (e.g., Seurat v3 integrated matrix with 2000 highly variable genes) as AnnData with labels and batch indicators in metadata
- An unlabeled query scRNA-seq dataset aligned to the reference space (e.g., PBMC-Bench10Xv3)

## Outputs

- Hierarchical classification tree with a trained linear SVM at each node
- Predicted cell-type labels for every cell of the query dataset
- Learned tree figure and a confusion matrix reflecting hierarchy boundaries

## How to apply

First align the annotated training datasets (e.g., PBMC-eQTL, PBMC-Bench10Xv2, PBMC-FACS integrated with Seurat v3 using 2000 highly variable genes) since scHPL is not a batch correction tool, and copy the integrated expression matrix into the .X slot of an AnnData object with labels and batch indicators as metadata columns. Run scHPL's learn function on the aligned datasets so it automatically matches cell populations across datasets and constructs the hierarchical classification tree. Choose the linear SVM as the node classifier: scHPL trains either a linear SVM, kNN, or one-class SVM at each node, and the linear SVM is advised for Seurat-integrated data that retains many dimensions; it matched or exceeded one-class SVM accuracy on all datasets, and the one-class SVM's performance dropped more on small populations. Apply the trained tree with scHPL's predict function to label the query dataset's cells, then use scHPL's evaluate module to inspect the learned tree and a confusion matrix that reflects the hierarchy boundaries.

## Related tools

- **scHPL** (Learns the hierarchical classification tree and trains the linear SVM at each node; its predict and evaluate functions label query cells and produce the tree figure and confusion matrix) — https://github.com/lcmmichielsen/scHPL
- **Seurat** (Used to batch-align the training datasets (v3 integration on 2000 highly variable genes), producing the high-dimensional integrated matrix for which the linear SVM is the advised node classifier)
- **scArches** (Advised alternative alignment tool before running scHPL, within the treeArches framework) — https://github.com/theislab/scarches

## Examples

```
pip install scHPL
```

## Evaluation signals

- Predicted labels of the held-out PBMC-Bench10Xv3 dataset reproduce the paper's results: most cells correctly annotated, with the learned tree and confusion matrix matching Figs. 5B and 5C
- Observed misclassifications (e.g., CD16+ monocytes vs mDC) are explainable by mislabeling in the training datasets, verified via marker gene expression
- Linear SVM accuracy is equal to or higher than the one-class SVM across datasets, especially on populations with a small number of cells
- The learned tree preserves the original annotations of the training datasets while correctly matching populations across the three PBMC datasets
- Datasets were aligned (e.g., via Seurat v3 with 2000 HVGs) before running scHPL, as required by the method

## Limitations

- Both linear and one-class SVMs show decreased performance on populations with a small number of cells, though the one-class SVM is more affected
- When a new dataset is added, the alignment — and consequently scHPL and its node classifiers — has to be rerun with current batch correction tools
- If datasets are not well aligned, it is hard to resolve whether mistakes in the constructed tree stem from erroneous alignment or mismatches created by scHPL
- The hierarchical-relationship assumption is inadequate in situations such as dedifferentiation (e.g., beta to alpha cell conversions in type 2 diabetes)

## Evidence

- [methods] Per-node classifier choice: "For each node in the tree either a linear SVM, kNN, or one-class SVM is trained."
- [methods] Prediction usage: "The trained classification tree can be used to predict the labels of a new unlabeled dataset."
- [discussion] Classifier comparison on small populations: "For both classifiers, we saw a decrease in performance on populations with a small number of cells, but for the one-class SVM this effect was more apparent."
- [readme] Alignment requirement: "NOTE: scHPL is not a batch correction tool, we advise to align the datasets before matching the cell populations."
