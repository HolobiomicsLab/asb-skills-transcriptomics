---
name: low-count-barcodes-recovery-filtering
description: Use when a droplet-based scRNA-seq raw unfiltered count matrix (h5 or
  mtx) has been filtered by a total-count cutoff and you suspect real low-count cells
  (e.g., platelets) were discarded.
when_to_use_negative:
- The count matrix is already filtered or only contains barcodes passing the original
  threshold — recovered below-threshold barcodes cannot be extracted without raw unfiltered
  data
- You need ambient RNA-corrected expression values — EmptyNN does not estimate corrected
  gene expression profiles; tools like SoupX are required for that
- Barcodes have total counts <= 10 — these are excluded from the EmptyNN analysis
  by design
edam_operation: http://edamontology.org/operation_3933
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: EmptyNN
  role: Positive-unlabeled-learning neural network that classifies barcodes as cell-free
    or cell-containing and recovers barcodes below the original filtering threshold
  repo: https://github.com/lkmklsmn/empty_nn
- name: Seurat
  role: 'Downstream processing of recovered barcodes: mitochondrial-fraction filtering,
    normalization, HVG detection, PCA, clustering (resolution = 0.2), and t-SNE visualization'
- name: Cell Ranger v.2 (10X Genomics)
  role: Source of the original total-count filtering threshold (200 in the cell hashing
    study) whose discarded low-count barcodes are being recovered
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.patter.2021.100311
    title: 'EmptyNN: A neural network based on positive and unlabeled learning to
      remove cell-free droplets and recover lost cells in scRNA-seq data'
  source_content_hash: sha256:af514aea2edcab4ff9ab392c3d780a507659dc070cd126ebb08abcda8c84751b
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
  - Cell Ranger v.2 (10X Genomics)
  - EmptyNN
  - Seurat
  edam_operation: http://edamontology.org/operation_3933
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# low-count-barcodes-recovery-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Recover and validate genuine cell-containing barcodes that fall below an original study's total-count filtering threshold (e.g., Cell Ranger v.2-style cutoff of 200) using EmptyNN's positive-unlabeled neural network, then characterize the recovered low-count cells with Seurat clustering and reference-dataset correlation.

## When to use

Use when a droplet-based scRNA-seq raw unfiltered count matrix (h5 or mtx) has been filtered by a total-count cutoff and you suspect real low-count cells (e.g., platelets) were discarded.

## When NOT to use

- The count matrix is already filtered or only contains barcodes passing the original threshold — recovered below-threshold barcodes cannot be extracted without raw unfiltered data
- You need ambient RNA-corrected expression values — EmptyNN does not estimate corrected gene expression profiles; tools like SoupX are required for that
- Barcodes have total counts <= 10 — these are excluded from the EmptyNN analysis by design

## Inputs

- Raw unfiltered scRNA-seq count matrix in h5 or mtx format (e.g., cell hashing dataset from Stoeckius et al.)
- Original study's total-count filtering threshold (e.g., Cell Ranger v.2-style threshold of 200)
- Independent reference scRNA-seq dataset for cluster identity validation (e.g., Reference PBMC 3k dataset)

## Outputs

- Boolean vector of cell-free/cell-containing predictions and probabilities for each barcode in set U
- Set of recovered low-count barcodes below the original threshold (e.g., 885 barcodes with counts 51–196, median 93.5 for platelets)
- t-SNE embedding of recovered barcodes (e.g., tsne_recovered_barcodes.png) resolving five distinct cell-type clusters
- Pearson correlation (Rho) between recovered platelet cluster and PBMC 3k platelet reference profile

## How to apply

Load the raw unfiltered cell hashing count matrix with Read10X_h5 and transpose it so rows are barcodes and columns are genes, excluding barcodes with total counts <= 10 from analysis. Run EmptyNN's emptynn() with default parameters (threshold T = 100, k = 10 cross-validation folds, 10 iterations) to classify each barcode as cell-free or cell-containing. Extract the retained barcodes whose total counts fall below the original study's threshold (200 total counts in the cell hashing case, yielding 885 recovered barcodes). Process the recovered barcode matrix in Seurat via runSeurat (resolution = 0.2) with mitochondrial-fraction filtering, normalization, HVG detection, PCA, clustering, and t-SNE. Assign cell-type identities via differential expression analysis and validate identity by computing the Pearson correlation between a recovered cluster (e.g., platelets) and a reference profile from an independent PBMC 3k dataset.

## Related tools

- **EmptyNN** (Positive-unlabeled-learning neural network that classifies barcodes as cell-free or cell-containing and recovers barcodes below the original filtering threshold) — https://github.com/lkmklsmn/empty_nn
- **Seurat** (Downstream processing of recovered barcodes: mitochondrial-fraction filtering, normalization, HVG detection, PCA, clustering (resolution = 0.2), and t-SNE visualization)
- **Cell Ranger v.2 (10X Genomics)** (Source of the original total-count filtering threshold (200 in the cell hashing study) whose discarded low-count barcodes are being recovered)

## Examples

```
counts <- t(Read10X_h5("./data/raw.h5")); nn.res <- emptynn(counts, threshold = 100, k = 10, iteration = 10); retained <- runSeurat(counts = counts[, nn.res$nn.keep], resolution = 0.2)
```

## Evaluation signals

- The t-SNE of recovered low-total-count barcodes resolves five distinct clusters matching the reported result
- Pearson correlation between the recovered platelet cluster and the PBMC 3k platelet reference reaches Rho = 0.95 (p < 2.2 × 10−6)
- Recovered barcodes have total counts below the original study's threshold of 200 (e.g., platelet counts range 51–196, median 93.5) and above the <= 10 exclusion floor
- The fraction of classified barcodes is consistent with the reported split: 20,833 (52.3%) cell-free and 19,009 (47.7%) cell-containing of 39,842 barcodes in the cell hashing dataset

## Limitations

- The key assumption that low total-count barcodes are cell-free may not hold when genuine cell-containing droplets with very low total counts exist; the user can adjust the T threshold to mitigate potential bias
- Parameter selection (number of cross-validation folds k and threshold T) must be specified manually; no hyperparameter selection approach is implemented
- Retained cell-containing droplets may have a high fraction of mitochondrial reads; low-quality cells passing initial filtering must be removed in downstream analysis

## Evidence
