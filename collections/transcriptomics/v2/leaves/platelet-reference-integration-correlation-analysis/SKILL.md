---
name: platelet-reference-integration-correlation-analysis
description: Use when you have recovered low-total-count barcodes (below an original
  Cell Ranger v.2-style threshold of 200) and need to confirm that a recovered cluster
  represents genuine cells of a known type rather than cell-free droplets.
when_to_use_negative:
- The goal is ambient RNA correction of gene expression estimates — EmptyNN does not
  estimate corrected expression profiles; use a tool such as SoupX instead
- The dataset lacks a trustworthy independent reference profile for the candidate
  cell type, making correlation-based identity confirmation impossible
- Barcodes have total counts <= 10, which are excluded from the EmptyNN analysis entirely
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: EmptyNN
  role: Cell-calling via positive-unlabeled learning; recovers barcodes below the
    original filtering threshold that are then validated as genuine cells
  repo: https://github.com/lkmklsmn/empty_nn
- name: Seurat
  role: Downstream processing (mitochondrial-fraction filtering, normalization, HVG,
    PCA, clustering at resolution 0.2, t-SNE) of the recovered barcodes
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.patter.2021.100311
    title: 'EmptyNN: A neural network based on positive and unlabeled learning to
      remove cell-free droplets and recover lost cells in scRNA-seq data'
  source_content_hash: sha256:3b0d7a615426287c6cb40af2a48ef65ad7625c3830b967ad5bc688358c26be97
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
  - EmptyNN
  - Seurat
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# platelet-reference-integration-correlation-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates that a low-count cell cluster recovered by EmptyNN from the cell hashing dataset is genuinely platelets by computing the Pearson correlation between the recovered cluster's expression profile and a platelet reference profile derived from an independent PBMC 3k dataset.

## When to use

Use when you have recovered low-total-count barcodes (below an original Cell Ranger v.2-style threshold of 200) and need to confirm that a recovered cluster represents genuine cells of a known type rather than cell-free droplets.

## When NOT to use

- The goal is ambient RNA correction of gene expression estimates — EmptyNN does not estimate corrected expression profiles; use a tool such as SoupX instead
- The dataset lacks a trustworthy independent reference profile for the candidate cell type, making correlation-based identity confirmation impossible
- Barcodes have total counts <= 10, which are excluded from the EmptyNN analysis entirely

## Inputs

- Raw unfiltered cell hashing count matrix (h5 or mtx format), transposed to barcodes x genes
- EmptyNN cell-calling predictions (nn.keep boolean vector and probabilities per barcode in set U)
- Recovered barcodes with total counts < 200 and > 10 (885 barcodes)
- Independent reference PBMC 3k scRNA-seq dataset

## Outputs

- t-SNE embedding of recovered barcodes separating five distinct cell-type clusters (tsne_recovered_barcodes.png)
- Platelet cell-type assignment for the recovered cluster via differential expression analysis
- Pearson correlation (Rho) between the recovered platelet cluster profile and the PBMC 3k platelet reference profile, with p-value

## How to apply

After running EmptyNN on the raw unfiltered cell hashing count matrix (loaded with Read10X_h5 and transposed so rows are barcodes and columns are genes), extract the 885 recovered barcodes whose total counts fall below the original study's Cell Ranger v.2-style threshold of 200, excluding barcodes with total counts <= 10. Process the recovered barcode count matrix in Seurat via runSeurat (resolution = 0.2), including mitochondrial-fraction filtering, normalization, HVG detection, PCA, clustering, and t-SNE, which should resolve five unique clusters. Assign cell-type identities by differential expression analysis to identify the platelet cluster (recovered platelets have total counts in the range 51–196, median 93.5). Integrate the independent reference PBMC 3k dataset to obtain its platelet reference profile, then compute the Pearson correlation between the recovered platelet cluster and this reference profile; acceptance requires Rho = 0.95 with p < 2.2 × 10−6, matching the reported result.

## Related tools

- **EmptyNN** (Cell-calling via positive-unlabeled learning; recovers barcodes below the original filtering threshold that are then validated as genuine cells) — https://github.com/lkmklsmn/empty_nn
- **Seurat** (Downstream processing (mitochondrial-fraction filtering, normalization, HVG, PCA, clustering at resolution 0.2, t-SNE) of the recovered barcodes)

## Examples

```
library(EmptyNN); counts <- t(Read10X_h5("data/cell_hashing_raw.h5")); nn.res <- emptynn(counts, threshold=100, k=10, iteration=10); retained <- runSeurat(counts = counts[, nn.res$nn.keep], resolution = 0.2)
```

## Evaluation signals

- The t-SNE of recovered low-total-count barcodes resolves five distinct clusters
- Differential expression analysis identifies a platelet cluster among the five recovered clusters
- Recovered platelet total counts fall in the range 51–196 (median 93.5), below the original threshold of 200
- Pearson correlation between the recovered platelet cluster and the PBMC 3k platelet reference reaches Rho = 0.95
- Correlation p-value is below 2.2 × 10−6

## Limitations

- The correlation is only meaningful if a reliable platelet reference profile exists; it depends on integration with the external PBMC 3k dataset
- EmptyNN does not correct ambient RNA contamination, so expression profiles used in the correlation may still carry ambient RNA effects unless SoupX is applied
- The validation addresses only the platelet cluster; other recovered clusters are validated indirectly by t-SNE separation and DE-based annotation

## Evidence

- [results] we integrated an independent dataset profiling peripheral blood mononuclear cells (PBMCs) (see details in "Reference PBMC 3k dataset": [evidence span withheld — anchor status: not_found]
