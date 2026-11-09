---
name: single-cell-differential-expression-analysis
description: Use when you have partitioned droplet-based scRNA-seq barcodes into cell-calling
  groups (e.g. All-retained, EmptyNN-retained, EmptyNN-removed) and need to test whether
  a cell-caller's retained barcodes are transcriptionally similar to confirmed cells.
when_to_use_negative:
- Input is already filtered/normalized expression (e.g. a Seurat object with a subset
  of cells) — the contrasts require the raw unfiltered matrix so barcodes below the
  200-count threshold can be partitioned
- Goal is cell-type annotation of recovered clusters via marker genes rather than
  validating cell-calling agreement between barcode groups — the downsampling design
  here is specific to comparing EmptyNN-retained vs EmptyNN-removed groups
- No cell-calling labels (e.g. nn.keep vector) exist for the barcodes, so the three
  group partition cannot be constructed
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: Seurat
  role: Builds Seurat objects for downsampled barcode groups and runs FindMarkers()
    to compute differential expression between All-retained, EmptyNN-retained, and
    EmptyNN-removed groups
  repo: https://github.com/lkmklsmn/empty_nn
- name: EmptyNN
  role: Provides the nn.keep classification that partitions barcodes into EmptyNN-retained
    and EmptyNN-removed groups prior to the DE contrasts
  repo: https://github.com/lkmklsmn/empty_nn
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.patter.2021.100311
    title: 'EmptyNN: A neural network based on positive and unlabeled learning to
      remove cell-free droplets and recover lost cells in scRNA-seq data'
  source_content_hash: sha256:729831b78583258fad4bd400206ef98730cadf8b98ec436426833db8382e976d
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
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# single-cell-differential-expression-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Differential expression analysis in Seurat comparing barcodes retained by a cell-calling method (All-retained) against barcodes removed (EmptyNN-removed) or specifically recovered (EmptyNN-retained), after downsampling UMI counts and cells to remove group-size and depth bias, to validate that recovered barcodes resemble genuine cells.

## When to use

Use when you have partitioned droplet-based scRNA-seq barcodes into cell-calling groups (e.g. All-retained, EmptyNN-retained, EmptyNN-removed) and need to test whether a cell-caller's retained barcodes are transcriptionally similar to confirmed cells.

## When NOT to use

- Input is already filtered/normalized expression (e.g. a Seurat object with a subset of cells) — the contrasts require the raw unfiltered matrix so barcodes below the 200-count threshold can be partitioned
- Goal is cell-type annotation of recovered clusters via marker genes rather than validating cell-calling agreement between barcode groups — the downsampling design here is specific to comparing EmptyNN-retained vs EmptyNN-removed groups
- No cell-calling labels (e.g. nn.keep vector) exist for the barcodes, so the three group partition cannot be constructed

## Inputs

- Raw unfiltered cell hashing count matrix (h5 or mtx format)
- EmptyNN classification results (nn.keep boolean vector from emptynn(counts, threshold = 100, k = 10, iteration = 10))
- Group partition of barcodes: All-retained (total counts >= 200), EmptyNN-retained, EmptyNN-removed

## Outputs

- DE gene table for All-retained vs EmptyNN-removed (de_all_vs_removed.csv), filtered at adjusted p < 0.01
- DE gene table for All-retained vs EmptyNN-retained (de_all_vs_retained.csv), filtered at adjusted p < 0.01
- Count of significant DE genes per contrast used as a transcriptional-similarity metric

## How to apply

Partition barcodes into All-retained (passing the original study's Cell Ranger v.2 threshold of 200 total counts), EmptyNN-retained (kept by emptynn(counts, threshold = 100, k = 10, iteration = 10)), and EmptyNN-removed barcodes from the raw unfiltered cell hashing count matrix. To remove bias from unbalanced numbers of cells and total UMI count between groups, downsample both counts and cells before comparison. Build Seurat objects for the downsampled groups and run FindMarkers() for the contrasts All-retained vs EmptyNN-removed and All-retained vs EmptyNN-retained. Filter results at adjusted p-value < 0.01 and compare the number of significant DE genes: far fewer DE genes for All-retained vs EmptyNN-retained than All-retained vs EmptyNN-removed indicates that EmptyNN-retained barcodes resemble genuine cell-containing droplets.

## Related tools

- **Seurat** (Builds Seurat objects for downsampled barcode groups and runs FindMarkers() to compute differential expression between All-retained, EmptyNN-retained, and EmptyNN-removed groups) — https://github.com/lkmklsmn/empty_nn
- **EmptyNN** (Provides the nn.keep classification that partitions barcodes into EmptyNN-retained and EmptyNN-removed groups prior to the DE contrasts) — https://github.com/lkmklsmn/empty_nn

## Examples

```
nn.res <- emptynn(counts, threshold = 100, k = 10, iteration = 10); markers <- FindMarkers(seurat_obj, ident.1 = "all_retained", ident.2 = "emptynn_removed"); de <- subset(markers, p_val_adj < 0.01); write.csv(de, "de_all_vs_removed.csv")
```

## Evaluation signals

- In CD4 T cells, the All-retained vs EmptyNN-removed contrast should yield approximately 3,898 significant DE genes at adjusted p < 0.01
- In CD4 T cells, the All-retained vs EmptyNN-retained contrast should yield approximately 155 significant DE genes at adjusted p < 0.01 — roughly 25-fold fewer than the removed contrast
- Group sizes and total UMI counts must be balanced across contrasts after downsampling (verify totals match before running FindMarkers())
- Both DE tables must be produced as de_all_vs_removed.csv and de_all_vs_retained.csv with adjusted p-value columns for filtering

## Limitations

- Downsampling is required because unbalanced numbers of cells and total UMI counts between groups would otherwise bias DE results
- EmptyNN does not correct ambient RNA contamination in expression estimates; apply SoupX if corrected profiles are needed before DE
- EmptyNN-retained droplets may have high mitochondrial read fractions and should be filtered separately in downstream analysis
- Biological interpretation of differences between cell-calling methods may be limited where classifications overlap heavily

## Evidence
