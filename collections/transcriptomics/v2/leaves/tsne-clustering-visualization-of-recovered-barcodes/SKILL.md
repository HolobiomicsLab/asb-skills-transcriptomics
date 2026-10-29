---
name: tsne-clustering-visualization-of-recovered-barcodes
description: Use when you have EmptyNN-retained barcodes (e.g., 885 recovered barcodes
  below a Cell Ranger v.2 cutoff of 200 total counts) and need to visualize and identify
  their cell-type clusters via t-SNE.
when_to_use_negative:
- The count matrix is already filtered (e.g., Cell Ranger v.3 output) so low-count
  recovered barcodes are absent — EmptyNN recovery and this downstream visualization
  require the raw unfiltered matrix
- Barcodes with total counts <= 10 dominate the matrix, since EmptyNN excludes them
  from analysis before recovery
- The goal is ambient RNA correction rather than cell-calling visualization — EmptyNN
  does not correct gene expression estimates (use SoupX separately)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: EmptyNN
  role: Provides the recovered/retained barcodes (nn.keep vector) below the 200-count
    threshold that are then clustered and visualized
  repo: https://github.com/lkmklsmn/empty_nn
- name: Seurat
  role: Runs the downstream pipeline (mitochondrial filtering, normalization, HVG,
    PCA, clustering, t-SNE) via runSeurat at resolution = 0.2
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.patter.2021.100311
    title: 'EmptyNN: A neural network based on positive and unlabeled learning to
      remove cell-free droplets and recover lost cells in scRNA-seq data'
  source_content_hash: sha256:a441e1434d273bb98fdd091ca1694c2739f49e721c297056271b6c4f0f8e39fa
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# tsne-clustering-visualization-of-recovered-barcodes

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Seurat-based downstream processing (mitochondrial-fraction filtering, normalization, HVG detection, PCA, clustering at resolution 0.2, t-SNE) of barcodes recovered by EmptyNN below the original Cell Ranger v.2 threshold, used to resolve and annotate cell-type structure among low-total-count barcodes.

## When to use

Use when you have EmptyNN-retained barcodes (e.g., 885 recovered barcodes below a Cell Ranger v.2 cutoff of 200 total counts) and need to visualize and identify their cell-type clusters via t-SNE.

## When NOT to use

- The count matrix is already filtered (e.g., Cell Ranger v.3 output) so low-count recovered barcodes are absent — EmptyNN recovery and this downstream visualization require the raw unfiltered matrix
- Barcodes with total counts <= 10 dominate the matrix, since EmptyNN excludes them from analysis before recovery
- The goal is ambient RNA correction rather than cell-calling visualization — EmptyNN does not correct gene expression estimates (use SoupX separately)

## Inputs

- Raw unfiltered scRNA-seq count matrix (h5 or mtx format), transposed so rows are barcodes and columns are genes
- EmptyNN predictions: boolean keep vector and probabilities for barcodes in set U
- Independent reference dataset (PBMC 3k) for cell-type validation

## Outputs

- Seurat object with clustered recovered barcodes and t-SNE embedding (e.g., tsne_recovered_barcodes.png / DimPlot)
- Cell-type annotations for the five recovered clusters from differential expression analysis
- Pearson correlation statistic against the reference platelet profile (Rho = 0.95)

## How to apply

Take the raw unfiltered count matrix (h5 or mtx) transposed so rows are barcodes and columns are genes, and subset to the EmptyNN-retained barcodes (nn.keep), excluding barcodes with total counts <= 10. Run Seurat's runSeurat with resolution = 0.2, which applies filtering of low-quality cells by mitochondrial fraction, normalization, highly variable gene detection, PCA, clustering, and t-SNE embedding. Confirm the t-SNE resolves five distinct clusters among the recovered low-total-count barcodes, then assign cell-type identities by differential expression analysis. Finally, validate annotation by integrating an independent reference dataset (e.g., the PBMC 3k platelet reference) and computing a Pearson correlation between the recovered platelet cluster and the reference profile (expected Rho = 0.95, p < 2.2e-6).

## Related tools

- **EmptyNN** (Provides the recovered/retained barcodes (nn.keep vector) below the 200-count threshold that are then clustered and visualized) — https://github.com/lkmklsmn/empty_nn
- **Seurat** (Runs the downstream pipeline (mitochondrial filtering, normalization, HVG, PCA, clustering, t-SNE) via runSeurat at resolution = 0.2)

## Examples

```
retained <- runSeurat(counts = counts[, nn.res$nn.keep], resolution = 0.2); DimPlot(retained, reduction = 'tsne')
```

## Evaluation signals

- The t-SNE of recovered barcodes resolves five distinct clusters, matching the reported figure (tsne_recovered_barcodes.png)
- The recovered platelet cluster's expression profile correlates with the PBMC 3k platelet reference at Rho = 0.95 (p < 2.2e-6)
- Recovered barcodes' total counts fall below the original filtering threshold of 200 (recovered platelets: range 51-196, median 93.5)
- Approximately 885 barcodes are recovered below the threshold, with barcodes of total counts <= 10 excluded
- The pipeline applies identical processing: mitochondrial-fraction filtering, normalization, HVG detection, PCA, clustering, t-SNE

## Limitations

- Retained cell-containing droplets may have a high fraction of mitochondrial reads; low-quality cells that pass initial filtering require separate removal in downstream analysis
- Parameter selection (clustering resolution, cross-validation folds k, threshold T) is manual; EmptyNN implements no hyperparameter selection approach
- EmptyNN does not correct for ambient RNA contamination, so gene expression estimates in the t-SNE/clusters may reflect ambient RNAs unless SoupX is applied

## Evidence
