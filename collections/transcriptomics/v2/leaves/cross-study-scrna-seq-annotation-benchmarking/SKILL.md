---
name: cross-study-scrna-seq-annotation-benchmarking
description: Use when you have scRNA-seq data with known labels from one study and
  need to annotate cells from a different, independently collected study of the same
  tissue (inter-study annotation), or to benchmark label-assignment methods.
when_to_use_negative:
- The new sample contains cell types (e.g., malignant or novel cells) not present
  in the reference/training data — NeuCA is designed for settings where all cell types
  already exist in the training data
- No labeled training data exist for the tissue type of interest — unsupervised clustering
  is the only viable option in that case
- Unknown cell types are expected but not a priori identifiable — pre-screen with
  tools for rare/unknown cell identification or expert consultation before applying
  NeuCA
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: NeuCA
  role: 'Primary supervised classifier: correlation-based strategy selection (tau
    = 0.95) between feed-forward and hierarchical neural networks for label assignment'
  repo: https://bioconductor.org/packages/NeuCA
- name: scmap
  role: Competing label-assignment method (scmap-cluster and scmap-cell) run on the
    same training data for accuracy comparison
- name: CHETAH
  role: Competing label-assignment method benchmarked against NeuCA in inter- and
    intra-study settings
- name: R
  role: R/Bioconductor environment hosting NeuCA, DuoClustering2018, and SingleCellExperiment
    data formatting
  repo: https://bioconductor.org/packages/NeuCA/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-021-04473-4
    title: A neural network-based method for exhaustive cell label assignment using
      single cell RNA-seq data
  source_content_hash: sha256:5aedb4d42d95ce39a89bd5fea45c5501a349961e07cb39a3df9391a28b81910f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CHETAH
  - NeuCA
  - R
  - scmap
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# cross-study-scrna-seq-annotation-benchmarking

## Summary

Benchmark supervised cell label assignment (NeuCA vs scmap-cluster, scmap-cell, CHETAH) on FACS-sorted PBMC datasets by training on one study and testing on an independently collected one, quantifying annotation accuracy per method per train/test combination.

## When to use

Use when you have scRNA-seq data with known labels from one study and need to annotate cells from a different, independently collected study of the same tissue (inter-study annotation), or to benchmark label-assignment methods.

## When NOT to use

- The new sample contains cell types (e.g., malignant or novel cells) not present in the reference/training data — NeuCA is designed for settings where all cell types already exist in the training data
- No labeled training data exist for the tissue type of interest — unsupervised clustering is the only viable option in that case
- Unknown cell types are expected but not a priori identifiable — pre-screen with tools for rare/unknown cell identification or expert consultation before applying NeuCA

## Inputs

- Normalized scRNA-seq count matrices as SingleCellExperiment objects with cell type labels in colData (normcounts assay)
- Zheng_8ct_PBMC dataset from R/Bioconductor package DuoClustering2018
- FACS-sorted NK cell dataset (GEO GSE130430)
- FACS-sorted monocytes dataset (GEO GSE103544)
- Training and testing matrices subset to overlapped genes

## Outputs

- benchmark_accuracy_table.csv with annotation accuracy per method per train/test combination
- Per-cell predicted label vectors from NeuCA, scmap-cluster, scmap-cell, and CHETAH

## How to apply

Load the FACS-sorted PBMC datasets — Zheng_8ct_PBMC from the R/Bioconductor package DuoClustering2018, the NK dataset from GEO GSE130430, and the monocytes dataset from GEO GSE103544 — normalize the expression matrices, and subset training and testing matrices to their overlapped genes. Format both as SingleCellExperiment objects with normcounts assays and feature_symbol in rowData, removing duplicated gene names. Train NeuCA on each training set using its correlation-based strategy selection: it computes averaged expression profiles per cell type, builds a Pearson correlation matrix, and applies threshold tau = 0.95 to choose between a feed-forward neural network and a marker-guided hierarchical neural network; run scmap-cluster, scmap-cell, and CHETAH on the same training data as competing label-assignment methods. Predict labels on the testing matrix with all four methods, compute accuracy per method per train/test combination (excluding FACmix as training), and write the comparison to benchmark_accuracy_table.csv. Correct application is verified when NeuCA reaches >98% accuracy in inter-study settings and outperforms existing methods intra-study.

## Related tools

- **NeuCA** (Primary supervised classifier: correlation-based strategy selection (tau = 0.95) between feed-forward and hierarchical neural networks for label assignment) — https://bioconductor.org/packages/NeuCA
- **scmap** (Competing label-assignment method (scmap-cluster and scmap-cell) run on the same training data for accuracy comparison)
- **CHETAH** (Competing label-assignment method benchmarked against NeuCA in inter- and intra-study settings)
- **R** (R/Bioconductor environment hosting NeuCA, DuoClustering2018, and SingleCellExperiment data formatting) — https://bioconductor.org/packages/NeuCA/

## Examples

```
predicted.label = NeuCA(train = Baron_sce, test = Seg_sce, model.size = "big", verbose = FALSE)
```

## Evaluation signals

- NeuCA inter-study annotation accuracy exceeds 98% on the FACS-sorted PBMC train/test combinations
- NeuCA accuracy exceeds scmap-cluster, scmap-cell, and CHETAH in intra-study predictions
- Training and testing matrices contain identical gene sets after subsetting to overlapped genes
- Confusion matrix off-diagonal counts are near zero (e.g., 328/331 alpha cells correct, only minor gamma-to-alpha confusion in the README example)
- benchmark_accuracy_table.csv contains one accuracy value per method per train/test combination, with FACmix excluded as training

## Limitations

- NeuCA is restricted to settings where all cell types in the new sample already exist in the reference/training data
- Cross-condition prediction may not hold in scenarios other than the ASD data application
- CHETAH produces an unignorable amount of unassigned/intermediate cell labels even when researchers expect a tiny portion or no unknown cell types
- No version numbers given for keras, TensorFlow, or R in the software description
- Deciding whether unknown/novel cell types exist in the data is not addressed by the method; expert consultation is recommended

## Evidence

- [results] NeuCA showed outstanding performance in inter-study annotation, correctly annotating the inter-study dataset with more than 98% accuracy.: [evidence span withheld — anchor status: not_found]
- [methods] Correlation-based strategy selection threshold: [evidence span withheld — anchor status: not_found]
- [methods] Datasets used: "The Zheng_8ct_PBMC data were downloaded from the R/Bioconductor package DuoClustering2018"
- [readme] README invocation: "predicted.label = NeuCA(train = Baron_sce, test = Seg_sce, 
                        model.size = "big", verbose = FALSE)"
- [readme] README input format: [evidence span withheld — anchor status: external_doc]
