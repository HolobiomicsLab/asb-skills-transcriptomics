---
name: chetah-hierarchical-cell-annotation
description: Use when benchmarking cell type annotation on scRNA-seq data with a labeled
  reference/training set, and you need CHETAH as a baseline label-assignment method
  to compare accuracy against neural-network or scmap-based classifiers.
when_to_use_negative:
- When no labeled reference/training data exist for the tissue of interest — supervised
  reference-based methods including CHETAH are inapplicable and unsupervised clustering
  (e.g., Seurat, SC3) is the only viable option
- When the test data contain cell types absent from the reference set, or unknown/rare
  cells are expected — CHETAH produces an unignorable amount of unassigned/intermediate
  cell labels in such settings, which is undesirable when exhaustive assignment is
  required
- When the goal is clustering de novo rather than transferring known labels to new
  data
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: CHETAH
  role: 'Primary method: hierarchy-based reference-supervised cell label assignment
    applied to train/test scRNA-seq matrices to produce predicted cell labels'
- name: R
  role: Implementation environment in which CHETAH is run alongside NeuCA, scmap-cluster,
    and scmap-cell for the benchmark
- name: NeuCA
  role: Supervised neural-network method being benchmarked against CHETAH in the same
    train/test pipeline
  repo: https://bioconductor.org/packages/NeuCA
- name: scmap
  role: Competing label-assignment method run alongside CHETAH on identical training
    data
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-021-04473-4
    title: A neural network-based method for exhaustive cell label assignment using
      single cell RNA-seq data
  source_content_hash: sha256:9135f85a15a5e9203687a3d6a8bcd9c920c0ad6be51fb848998c9c9e4d8d6c02
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CHETAH
  - NeuCA
  - R
  - scmap
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# chetah-hierarchical-cell-annotation

## Summary

Apply CHETAH, a correlation/hierarchy-based reference method for cell label assignment, as a competing annotation approach when benchmarking supervised label-assignment methods like NeuCA on scRNA-seq data with labeled training sets.

## When to use

Use when benchmarking cell type annotation on scRNA-seq data with a labeled reference/training set, and you need CHETAH as a baseline label-assignment method to compare accuracy against neural-network or scmap-based classifiers.

## When NOT to use

- When no labeled reference/training data exist for the tissue of interest — supervised reference-based methods including CHETAH are inapplicable and unsupervised clustering (e.g., Seurat, SC3) is the only viable option
- When the test data contain cell types absent from the reference set, or unknown/rare cells are expected — CHETAH produces an unignorable amount of unassigned/intermediate cell labels in such settings, which is undesirable when exhaustive assignment is required
- When the goal is clustering de novo rather than transferring known labels to new data

## Inputs

- Labeled training scRNA-seq expression matrix with known cell type labels (e.g., SingleCellExperiment or comparable matrix from Zheng_8ct_PBMC, GSE130430 NK cells, or GSE103544 monocytes)
- Unlabeled (or truth-held-out) testing scRNA-seq expression matrix from a different study or condition
- Overlapped gene set between training and testing matrices

## Outputs

- Per-cell predicted cell type labels for the testing dataset
- Annotation accuracy per method per train/test combination, written to benchmark_accuracy_table.csv

## How to apply

Run CHETAH on the same normalized training data used by the supervised method (e.g., NeuCA trained on Zheng_8ct_PBMC from DuoClustering2018, FACS-sorted NK cells from GSE130430, or FACS-sorted monocytes from GSE103544), then predict cell labels on the testing matrix subset to the overlapped genes between train and test. Train/test combinations are formed across independently collected PBMC datasets to test inter- and intra-study settings. Compute per-cell-type annotation accuracy (fraction of correctly assigned labels over all test cells) per method and per train/test combination, and compare against the competing methods' accuracy table. CHETAH is a reference-supervised, tree-based assignment method, so it consumes the labeled training matrix directly without neural network training; a correct application requires identical preprocessing (normalized expression matrices, common gene set) across all compared methods.

## Related tools

- **CHETAH** (Primary method: hierarchy-based reference-supervised cell label assignment applied to train/test scRNA-seq matrices to produce predicted cell labels)
- **R** (Implementation environment in which CHETAH is run alongside NeuCA, scmap-cluster, and scmap-cell for the benchmark)
- **NeuCA** (Supervised neural-network method being benchmarked against CHETAH in the same train/test pipeline) — https://bioconductor.org/packages/NeuCA
- **scmap** (Competing label-assignment method run alongside CHETAH on identical training data)

## Examples

```
predicted.label = NeuCA(train = Baron_sce, test = Seg_sce, model.size = "big", verbose = FALSE)
```

## Evaluation signals

- Per-cell predicted labels are exhaustively assigned (no unassigned/intermediate labels expected when all test cell types exist in the reference)
- Annotation accuracy is computed as correctly assigned cells / total test cells, per train/test combination, and compared against scmap-cluster, scmap-cell, and NeuCA on the same gene-overlapped matrices
- Accuracy values land in the benchmark_accuracy_table.csv with one row per method × train/test combination
- Preprocessing is identical across methods: expression matrices normalized and both train and test subset to their overlapped genes before any method runs
- For inter-study PBMC combinations, the benchmark's reference point is that NeuCA achieves >98% accuracy, so CHETAH's accuracy should be reported relative to that bar

## Limitations

- CHETAH produces an unignorable amount of unassigned/intermediate cell labels even when researchers expect a tiny portion or no unknown cell types, which is not desirable when exhaustive label assignment is required
- Correlation/hierarchy-based methods including CHETAH may have difficulty accurately assigning labels when pairwise correlations of several cell types are high simultaneously

## Evidence

- [intro] Popular methods in this category include scmap13, CHETAH14, CellAssign15, and others: "Popular methods in this category include scmap13, CHETAH14, CellAssign15, and others"
- [intro] CHETAH unassigned labels: "CHETAH still produces an unignorable amount of unassigned/intermediate cell labels"
- [other] Run scmap-cluster, scmap-cell, and CHETAH as competing methods: [evidence span withheld — anchor status: not_found]
- [intro] High-correlation difficulty: "The methods in this category may have difficulty accurately assigning labels under these high correlation settings"
