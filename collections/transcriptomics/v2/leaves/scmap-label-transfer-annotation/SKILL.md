---
name: scmap-label-transfer-annotation
description: Use when annotating cells in a target scRNA-seq dataset by transferring
  labels from an independently collected, labeled reference dataset (e.g., inter-study
  PBMC annotation) and benchmarking against supervised methods.
when_to_use_negative:
- When pairwise correlations of several cell types in the reference are simultaneously
  high — correlation-based methods in this category may have difficulty accurately
  assigning labels under these high correlation settings
- When no labeled training/reference data exist for the tissue of interest — supervised/label-transfer
  methods require a reference, and unsupervised clustering is then the only viable
  solution
- When unknown or novel cell types are expected in the target data — label-assignment
  methods still label a considerable number of cells as unassigned, which is not desirable,
  and the target's cell types must already exist in the reference
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scmap
  role: Provides scmap-cluster and scmap-cell, the correlation-based label-assignment
    baseline trained on the reference PBMC data and applied to the inter-study testing
    matrix
- name: CHETAH
  role: Competing label-assignment method run on the same training data for the accuracy
    benchmark
- name: NeuCA
  role: Supervised neural-network classifier that scmap-based annotation is benchmarked
    against (>98% inter-study accuracy)
  repo: https://bioconductor.org/packages/NeuCA/
- name: R
  role: Implementation environment; scmap, CHETAH, and DuoClustering2018 are R/Bioconductor
    packages
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-021-04473-4
    title: A neural network-based method for exhaustive cell label assignment using
      single cell RNA-seq data
  source_content_hash: sha256:f8433afeb967c43de33f07dee821bb030f310a87ee5918f3587941afc9cc9591
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CHETAH
  - NeuCA
  - R
  - scmap
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# scmap-label-transfer-annotation

## Summary

Correlation-based label-transfer annotation of scRNA-seq cells with scmap-cluster and scmap-cell, using a labeled reference dataset to assign cell identities to a target dataset; used here as the established label-assignment baseline against NeuCA's neural-network classifier on FACS-sorted PBMC studies.

## When to use

Use when annotating cells in a target scRNA-seq dataset by transferring labels from an independently collected, labeled reference dataset (e.g., inter-study PBMC annotation) and benchmarking against supervised methods.

## When NOT to use

- When pairwise correlations of several cell types in the reference are simultaneously high — correlation-based methods in this category may have difficulty accurately assigning labels under these high correlation settings
- When no labeled training/reference data exist for the tissue of interest — supervised/label-transfer methods require a reference, and unsupervised clustering is then the only viable solution
- When unknown or novel cell types are expected in the target data — label-assignment methods still label a considerable number of cells as unassigned, which is not desirable, and the target's cell types must already exist in the reference

## Inputs

- Labeled training scRNA-seq expression matrix with known cell type labels (e.g., Zheng_8ct_PBMC, FACS-sorted NK GSE130430, FACS-sorted monocytes GSE103544)
- Testing scRNA-seq expression matrix from a different study/assay
- SingleCellExperiment-style count/normcounts matrices with feature symbols as row names

## Outputs

- Predicted cell type label per cell in the testing dataset
- Per-method, per-train/test-combination annotation accuracy values
- benchmark_accuracy_table.csv comparing scmap-cluster, scmap-cell, CHETAH, and NeuCA

## How to apply

Load a labeled training/reference dataset (e.g., Zheng_8ct_PBMC from the DuoClustering2018 R/Bioconductor package, or FACS-sorted NK cells from GEO GSE130430 and monocytes from GEO GSE103544) and an unlabeled testing matrix from a different study. Normalize both expression matrices and subset them to their overlapped genes, mirroring the preprocessing NeuCA applies. Train scmap-cluster (correlation-based cluster projection) and scmap-cell (nearest-neighbor cell projection) on the same training data used to fit competing methods such as NeuCA (Pearson correlation of averaged cell type profiles, threshold tau = 0.95 deciding feed-forward vs. hierarchical networks) and CHETAH. Predict cell labels on the testing matrix, then compute annotation accuracy per method per train/test combination and write the full comparison to benchmark_accuracy_table.csv. Rationale: scmap represents the popular correlation-based label-assignment category, and the benchmark showed these methods are outperformed by supervised neural-network approaches when labeled training data are available.

## Related tools

- **scmap** (Provides scmap-cluster and scmap-cell, the correlation-based label-assignment baseline trained on the reference PBMC data and applied to the inter-study testing matrix)
- **CHETAH** (Competing label-assignment method run on the same training data for the accuracy benchmark)
- **NeuCA** (Supervised neural-network classifier that scmap-based annotation is benchmarked against (>98% inter-study accuracy)) — https://bioconductor.org/packages/NeuCA/
- **R** (Implementation environment; scmap, CHETAH, and DuoClustering2018 are R/Bioconductor packages)

## Evaluation signals

- Annotation accuracy per method per train/test combination computed against true testing labels; NeuCA's >98% inter-study accuracy is the reference point scmap baselines fall short of
- Confusion matrix of predicted vs. true testing cell labels (e.g., table(predicted.label, Seg_true_cell_label)) to localize misassignments
- Fraction of cells left unassigned/intermediate: label-assignment methods should not produce a considerable number of unassigned labels when unknown cell types are not expected
- Comparison restricted to combinations excluding the FACmix data as training, as specified in the benchmark workflow
- benchmark_accuracy_table.csv must contain rows for all four methods (scmap-cluster, scmap-cell, CHETAH, NeuCA) across all train/test combinations

## Limitations

- Correlation-based label assignment can fail when several cell types are highly correlated simultaneously in the reference
- Existing label-assignment methods label a considerable number of cells as unassigned even when unknown cell types are not expected in the target data
- In the benchmark, when training data are available, the improved accuracy, speed, and reproducibility of supervised methods easily outperform correlation-based ones

## Evidence

- [intro] Popular label-assignment methods: "Popular methods in this category include scmap13, CHETAH14, CellAssign15, and others"
- [intro] High-correlation failure mode: "The methods in this category may have difficulty accurately assigning labels under these high correlation settings"
- [abstract] Unassigned-label weakness: "when unknown cell types are not expected, existing methods still label a considerable number of cells as unassigned, which is not desirable"
- [intro] Supervised outperformance rationale: "when training data are available, the improved accuracy, speed, and reproducibility of supervised methods easily outperform unsupervised ones"
- [readme] Evaluation via confusion matrix: "If you have the true cell type labels for the testing dataset, you may evaluate the predictive performance by a confusion matrix"
