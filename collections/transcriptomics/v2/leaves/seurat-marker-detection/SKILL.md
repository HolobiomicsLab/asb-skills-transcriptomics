---
name: seurat-marker-detection
description: Use when comparing transcriptional profiles of barcode groups defined
  by different cell-calling methods (e.g., All-retained vs EmptyNN-retained vs EmptyNN-removed)
  from a raw scRNA-seq count matrix.
when_to_use_negative:
- Groups have unbalanced numbers of cells or total UMI counts and downsampling has
  not been performed — the comparison will be biased
- Barcodes have not been partitioned into All-retained, EmptyNN-retained, and EmptyNN-removed
  groups (e.g., only a single filtered Seurat object exists)
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Seurat
  role: Build Seurat objects for the downsampled barcode groups and run FindMarkers()
    for the All-retained vs EmptyNN-removed and All-retained vs EmptyNN-retained contrasts
  repo: https://github.com/lkmklsmn/empty_nn
- name: EmptyNN
  role: Provides nn.keep vector from emptynn(counts, threshold = 100, k = 10, iteration
    = 10) used to define the EmptyNN-retained and EmptyNN-removed barcode groups being
    contrasted
  repo: https://github.com/lkmklsmn/empty_nn
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.patter.2021.100311
    title: 'EmptyNN: A neural network based on positive and unlabeled learning to
      remove cell-free droplets and recover lost cells in scRNA-seq data'
  source_content_hash: sha256:409b10ca586166a1f4a8ddee1c75c1f23d6857900d15ad91cd1f9acac125f7fb
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# seurat-marker-detection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Differential expression analysis with Seurat's FindMarkers() to test whether EmptyNN-retained barcodes are transcriptionally closer to genuine cells (All-retained) than EmptyNN-removed barcodes, using adjusted p < 0.01 as the significance filter.

## When to use

Use when comparing transcriptional profiles of barcode groups defined by different cell-calling methods (e.g., All-retained vs EmptyNN-retained vs EmptyNN-removed) from a raw scRNA-seq count matrix.

## When NOT to use

- Groups have unbalanced numbers of cells or total UMI counts and downsampling has not been performed — the comparison will be biased
- Barcodes have not been partitioned into All-retained, EmptyNN-retained, and EmptyNN-removed groups (e.g., only a single filtered Seurat object exists)

## Inputs

- Raw unfiltered cell hashing count matrix in h5 or mtx format
- EmptyNN classification results (nn.keep boolean vector)
- Original study's cell-calling threshold (e.g., 200 total counts) for defining All-retained barcodes

## Outputs

- de_all_vs_removed.csv — significant DE genes (adjusted p < 0.01) for All-retained vs EmptyNN-removed
- de_all_vs_retained.csv — significant DE genes (adjusted p < 0.01) for All-retained vs EmptyNN-retained

## How to apply

Load the raw unfiltered cell hashing count matrix (h5/mtx) and the EmptyNN classification (nn.keep from emptynn(counts, threshold = 100, k = 10, iteration = 10)), then partition barcodes into All-retained (passing the original study's 200-total-count threshold), EmptyNN-retained, and EmptyNN-removed groups. Build Seurat objects per group and, because unbalanced cell numbers and total UMI counts bias marker detection, downsample counts and cells so group sizes and total UMIs are balanced before running FindMarkers(). Run two contrasts: All-retained vs EmptyNN-removed and All-retained vs EmptyNN-retained, filtering markers at adjusted p-value < 0.01. Rationale: many fewer DE genes against EmptyNN-retained (155) than against EmptyNN-removed (3,898) in CD4 T cells indicates EmptyNN-retained barcodes resemble genuine cells.

## Related tools

- **Seurat** (Build Seurat objects for the downsampled barcode groups and run FindMarkers() for the All-retained vs EmptyNN-removed and All-retained vs EmptyNN-retained contrasts) — https://github.com/lkmklsmn/empty_nn
- **EmptyNN** (Provides nn.keep vector from emptynn(counts, threshold = 100, k = 10, iteration = 10) used to define the EmptyNN-retained and EmptyNN-removed barcode groups being contrasted) — https://github.com/lkmklsmn/empty_nn

## Examples

```
de <- FindMarkers(seurat_obj, ident.1 = "All-retained", ident.2 = "EmptyNN-removed")
```

## Evaluation signals

- Number of significant DE genes at adjusted p < 0.01 for All-retained vs EmptyNN-removed should reproduce the reported 3,898 in CD4 T cells
- Number of significant DE genes at adjusted p < 0.01 for All-retained vs EmptyNN-retained should reproduce the reported 155 in CD4 T cells
- Contrast with more DE genes (All-retained vs EmptyNN-removed, 3,898) should exceed the All-retained vs EmptyNN-retained contrast by an order of magnitude
- Confirm downsampling balanced group sizes and total UMI counts between the two contrasts before interpreting marker counts
- Both output CSVs (de_all_vs_removed.csv, de_all_vs_retained.csv) exist and contain only markers passing adjusted p < 0.01

## Limitations

- Retained cell-containing droplets may have a high fraction of mitochondrial reads; low-quality cells may need separate removal in downstream analysis
- EmptyNN does not correct for ambient RNA contamination in gene expression estimates; external tools such as SoupX may be needed before DE analysis
- Results depend on EmptyNN hyperparameters (threshold T, folds k, iterations N) which must be specified manually

## Evidence
