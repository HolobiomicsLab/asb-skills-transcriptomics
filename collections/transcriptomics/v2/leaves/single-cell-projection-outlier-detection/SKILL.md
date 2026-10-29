---
name: single-cell-projection-outlier-detection
description: Use when query cells have been projected into a reference atlas embedding
  (e.g., via ProjectSVR's ProjectNewdata with a ref.umap reduction) and you must determine
  which mapped cells are well-placed versus low-quality outliers before downstream
  label transfer.
when_to_use_negative:
- Query cells contain novel cell states absent from the reference — fixed-model projections
  may be misleading and low mapQ may reflect novelty rather than technical failure;
  compare unsupervised clustering to mapping results first
- You have not yet projected the query into the reference UMAP space — the metric
  requires projected coordinates, not raw counts or an unprojected Seurat object
- Projecting across platforms (e.g., Smart-seq2 to 10x) where performance declines
  due to differing UMI distributions — interpret mapQ cautiously rather than as a
  clean outlier signal
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: ProjectSVR
  role: Provides AddProjQual to compute mean.knn.dist, null-distribution empirical
    p-values, and BH-adjusted mapQ.p.adj; also provides MapQCPlot for visualization
    and subset-based filtering of low-quality mapped cells
  repo: https://github.com/JarningGau/ProjectSVR
- name: Seurat
  role: Holds the projected query cells and stores mean.knn.dist / mapQ.p.val / mapQ.p.adj
    in cell metadata; subset() is used to apply mean.knn.dist and mapQ.p.adj cutoffs
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbaf586
    title: 'ProjectSVR: mapping single-cell RNA-seq data to reference atlases by supported
      vector regression'
  source_content_hash: sha256:40d4614b0cde51d2858c214f5986ebee8b11f6c585cb3c0d6cc4892891d0e264
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/jarninggau/ProjectSVR
  repo_url: https://github.com/JarningGau/ProjectSVR
  tools:
  - ProjectSVR
  - Seurat
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# single-cell-projection-outlier-detection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assesses the quality of query-cell projections onto a reference atlas UMAP using ProjectSVR's mean k-NN distance metric, with empirical p-values estimated from a null distribution of randomly sampled cells and Benjamini-Hochberg correction, to flag low-quality or outlier mapped cells.

## When to use

Use when query cells have been projected into a reference atlas embedding (e.g., via ProjectSVR's ProjectNewdata with a ref.umap reduction) and you must determine which mapped cells are well-placed versus low-quality outliers before downstream label transfer.

## When NOT to use

- Query cells contain novel cell states absent from the reference — fixed-model projections may be misleading and low mapQ may reflect novelty rather than technical failure; compare unsupervised clustering to mapping results first
- You have not yet projected the query into the reference UMAP space — the metric requires projected coordinates, not raw counts or an unprojected Seurat object
- Projecting across platforms (e.g., Smart-seq2 to 10x) where performance declines due to differing UMI distributions — interpret mapQ cautiously rather than as a clean outlier signal

## Inputs

- Query Seurat object with projected ref.umap reduction attached (e.g., from ProjectNewdata on a DISCO PBMC reference model)
- Parameters k (number of nearest neighbors, e.g., 20) and repeats (null-distribution sample count, e.g., 1e4)

## Outputs

- Per-cell metadata columns mean.knn.dist, mapQ.p.val, mapQ.p.adj
- Per-cell mapping-quality metrics table (map_quality_metrics.csv)
- MapQCPlot QC figure (mapqc_plot.png) with adjusted-p and mean.knn.dist cutoffs
- Filtered Seurat object retaining only well-mapped cells

## How to apply

After projecting query cells with ProjectNewdata and attaching the ref.umap reduction to the Seurat object, run ProjectSVR::AddProjQual(object, k = 20, repeats = 1e4) to build a k-NN network on the projected cells and compute per-cell mean.knn.dist. Statistical significance is derived by computing the mean distance of K randomly selected cells repeated n times (1e4 repeats) to construct a null distribution, from which empirical p-values and BH-adjusted mapQ.p.adj are estimated. Store mean.knn.dist, mapQ.p.val, and mapQ.p.adj in cell metadata, then visualize with MapQCPlot using an adjusted-p cutoff (e.g., p.adj.cutoff = 1e-3) and a mean.knn.dist cutoff (e.g., 1.5–1.6; the paper also uses 2.4 in another context). Filter with subset(seu.q, mean.knn.dist < 1.6) or subset(seu.q, mapQ.p.adj < 0.1) to drop bad projections before LabelTransfer.

## Related tools

- **ProjectSVR** (Provides AddProjQual to compute mean.knn.dist, null-distribution empirical p-values, and BH-adjusted mapQ.p.adj; also provides MapQCPlot for visualization and subset-based filtering of low-quality mapped cells) — https://github.com/JarningGau/ProjectSVR
- **Seurat** (Holds the projected query cells and stores mean.knn.dist / mapQ.p.val / mapQ.p.adj in cell metadata; subset() is used to apply mean.knn.dist and mapQ.p.adj cutoffs)

## Examples

```
seu.q <- ProjectSVR::AddProjQual(seu.q, k = 20, repeats = 1e4); seu.q <- subset(seu.q, mean.knn.dist < 1.6)
```

## Evaluation signals

- Per-cell metadata contains all three columns: mean.knn.dist, mapQ.p.val, and mapQ.p.adj
- Empirical p-values come from a null distribution of 1e4 random repeats of K randomly selected cells, with Benjamini-Hochberg correction applied (p.adj)
- Fraction of cells with mean.knn.dist > 1.5 should match the reported ~2.6% low-quality mapping fraction on the PBMC quick-start data
- After filtering (e.g., mean.knn.dist < 1.6 or mapQ.p.adj < 0.1), the retained Seurat object should contain only well-mapped cells ready for LabelTransfer

## Limitations

- If query cells contain novel states absent in the reference, fixed-model projections may be misleading; users must compare unsupervised clustering to mapping results to detect inconsistencies
- The mean.knn.dist cutoff is context-dependent — the paper uses different thresholds (1.6, 2.0, 2.4) across examples, so thresholds must be calibrated per dataset
- Performance declines when projecting across platforms (e.g., Smart-seq2 to 10x) due to differing UMI distributions and sequencing depth, which can inflate apparent outlier rates

## Evidence

- [methods] task_004 finding: [evidence span withheld — anchor status: not_found]
