---
name: scrna-seq-count-matrix-processing
description: Use when starting from a raw unfiltered gene-by-barcode count matrix
  (e.g. Read10X_h5 h5 or mtx) that must be transposed, cell-called (EmptyNN, threshold
  100), and processed in Seurat before clustering or benchmarking.
when_to_use_negative:
- The input is already a filtered, normalized feature-barcode matrix or an h5ad/Seurat
  object — EmptyNN requires the raw unfiltered count matrix
- The data are not droplet-based scRNA-seq; barcodes with total UMI counts <= 10 are
  excluded and the threshold T = 100 assumption (low-count barcodes are cell-free)
  does not apply
- The goal is ambient RNA correction itself — EmptyNN does not estimate corrected
  gene expression profiles; use SoupX instead
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: EmptyNN
  role: Cell-calling algorithm run on the transposed raw count matrix to classify
    cell-free vs cell-containing droplets (threshold 100, k = 10, iteration = 10)
  repo: https://github.com/lkmklsmn/empty_nn
- name: Seurat
  role: 'Downstream processing of retained barcodes via runSeurat: mitochondrial-fraction
    filtering, normalization, HVG detection, PCA, clustering (resolution 0.2), t-SNE'
- name: Cell Ranger v.2
  role: Source of the raw unfiltered gene-by-barcode count matrix (e.g. generated
    from the SRR5398237 BAM); its threshold of 200 total counts defines the original
    filtering baseline
- name: SoupX
  role: Optional post-hoc correction of ambient RNA contamination in expression estimates,
    since EmptyNN does not correct expression profiles
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_007
  source_papers:
  - doi: 10.1016/j.patter.2021.100311
    title: 'EmptyNN: A neural network based on positive and unlabeled learning to
      remove cell-free droplets and recover lost cells in scRNA-seq data'
  source_content_hash: sha256:cb756a7a81edce496c3831321fc90f9440cf914dafceaa72ccc545e93ceae5fe
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/lkmklsmn/empty_nn
  repo_url: https://github.com/lkmklsmn/empty_nn
  tools:
  - Cell Ranger v.2
  - CellBender
  - EmptyDrops
  - EmptyNN
  - Seurat
  - SoupX
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9130434782608695
n_claims: 23
---

# scrna-seq-count-matrix-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Load and prepare a raw unfiltered droplet scRNA-seq count matrix (mtx or h5) for cell-calling and downstream analysis with EmptyNN and Seurat, including transposition to barcode-by-gene orientation and mitochondrial-fraction filtering, normalization, HVG detection, PCA, clustering, and t-SNE.

## When to use

Use when starting from a raw unfiltered gene-by-barcode count matrix (e.g. Read10X_h5 h5 or mtx) that must be transposed, cell-called (EmptyNN, threshold 100), and processed in Seurat before clustering or benchmarking.

## When NOT to use

- The input is already a filtered, normalized feature-barcode matrix or an h5ad/Seurat object — EmptyNN requires the raw unfiltered count matrix
- The data are not droplet-based scRNA-seq; barcodes with total UMI counts <= 10 are excluded and the threshold T = 100 assumption (low-count barcodes are cell-free) does not apply
- The goal is ambient RNA correction itself — EmptyNN does not estimate corrected gene expression profiles; use SoupX instead

## Inputs

- Raw unfiltered droplet scRNA-seq count matrix in mtx or h5 format (e.g. neurons_900_raw.h5, cell hashing dataset)
- HTO counts or demuxlet-derived labels when ground-truth benchmarking is required
- Independent reference dataset (e.g. PBMC 3k) for cell-type confirmation of recovered clusters

## Outputs

- Transposed barcode-by-gene count matrix ready for EmptyNN
- Boolean vector of cell-free/cell-containing droplet predictions per barcode plus probabilities for set U barcodes
- Seurat object with normalized expression, PCA, clusters, and t-SNE embedding (resolution = 0.2) of retained barcodes
- Per-cluster cell-type identities via differential expression analysis

## How to apply

Load the raw unfiltered count matrix with Read10X_h5 (or the mtx equivalent) and transpose it so rows are barcodes/cells and columns are genes, since EmptyNN expects cell-by-gene orientation. Exclude barcodes with total counts <= 10 from the EmptyNN analysis, then run emptynn() with default parameters (threshold T = 100, k = 10 cross-validation folds, N = 10 iterations) to obtain the boolean cell-free/cell-containing predictions and probabilities for barcodes in set U. For downstream analysis, subset the matrix to EmptyNN-retained barcodes and pass it to runSeurat (resolution = 0.2), which applies mitochondrial-fraction filtering of low-quality cells, normalization, highly variable gene detection, PCA, clustering, and t-SNE. EmptyNN does not correct ambient RNA, so apply a tool such as SoupX afterward if corrected expression profiles are needed.

## Related tools

- **EmptyNN** (Cell-calling algorithm run on the transposed raw count matrix to classify cell-free vs cell-containing droplets (threshold 100, k = 10, iteration = 10)) — https://github.com/lkmklsmn/empty_nn
- **Seurat** (Downstream processing of retained barcodes via runSeurat: mitochondrial-fraction filtering, normalization, HVG detection, PCA, clustering (resolution 0.2), t-SNE)
- **Cell Ranger v.2** (Source of the raw unfiltered gene-by-barcode count matrix (e.g. generated from the SRR5398237 BAM); its threshold of 200 total counts defines the original filtering baseline)
- **SoupX** (Optional post-hoc correction of ambient RNA contamination in expression estimates, since EmptyNN does not correct expression profiles)

## Examples

```
library(EmptyNN); counts <- t(Read10X_h5('./data/neurons_900_raw.h5', use.names=TRUE, unique.features=TRUE)); nn.res <- emptynn(counts, threshold = 100, k = 10, iteration = 10, verbose = TRUE); retained <- runSeurat(counts = counts[, nn.res$nn.keep], resolution = 0.2)
```

## Evaluation signals

- EmptyNN classifies 20,833 of 39,842 barcodes (52.3%) as cell-free and 19,009 (47.7%) as cell-containing on the cell hashing dataset
- 885 recovered barcodes fall below the original Cell Ranger v.2 threshold of 200 total counts; barcodes with total counts <= 10 are excluded
- AUROC against HTO-derived singlet/doublet labels reaches ~94.73% (cell hashing) or ~96.30% (multiplexed PBMC, demuxlet labels)
- t-SNE of recovered low-count barcodes resolves five distinct cell-type clusters, with the platelet cluster correlating with the PBMC 3k platelet reference at Rho = 0.95, p < 2.2e-6

## Limitations

- The assumption that low total count barcodes are cell-free may not hold when genuine cell-containing droplets with very low total counts exist; the T threshold may need manual adjustment
- Parameter selection (cross-validation folds k and threshold T) must be specified manually; no hyperparameter selection approach is implemented
- Retained cell-containing droplets may have a high fraction of mitochondrial reads and require separate removal of low-quality cells in downstream analysis

## Evidence

- [results] Default EmptyNN parameters: [evidence span withheld — anchor status: not_found]
