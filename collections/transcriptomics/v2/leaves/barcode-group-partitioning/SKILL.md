---
name: barcode-group-partitioning
description: Use when validating a cell-calling filter (e.g., EmptyNN) by testing
  whether barcodes it uniquely retains are transcriptionally closer to confirmed cells
  than barcodes it removes, from a raw unfiltered count matrix.
when_to_use_negative:
- The input is already a filtered cell×gene matrix — the partition requires the raw
  unfiltered matrix so EmptyNN-removed low-count barcodes are present
- The goal is ambient RNA correction rather than barcode-group comparison — EmptyNN
  does not correct ambient RNA; tools such as SoupX are needed for that
- There is only a single cell-calling result with no competing filter definition (e.g.,
  no original-study threshold) to form the All-retained contrast group
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: EmptyNN
  role: Produces the nn.keep classification used to define EmptyNN-retained and EmptyNN-removed
    barcode groups from the raw count matrix
  repo: https://github.com/lkmklsmn/empty_nn
- name: Seurat
  role: Builds Seurat objects for downsampled barcode groups and runs FindMarkers()
    for the two differential expression contrasts
- name: Cell Ranger v.2
  role: Source of the original study's total-count filtering threshold (200) used
    to define the All-retained group
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.patter.2021.100311
    title: 'EmptyNN: A neural network based on positive and unlabeled learning to
      remove cell-free droplets and recover lost cells in scRNA-seq data'
  source_content_hash: sha256:422343093670653c4249db688c4a893b10e27595d0cf87653da90f0d0323583d
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
  - EmptyNN
  - Seurat
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# barcode-group-partitioning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Partitions raw scRNA-seq barcodes into All-retained (original Cell Ranger v.2 threshold of 200 total counts), EmptyNN-retained, and EmptyNN-removed groups, then contrasts them via downsampled Seurat FindMarkers() differential expression to test whether EmptyNN-retained barcodes resemble genuine cells.

## When to use

Use when validating a cell-calling filter (e.g., EmptyNN) by testing whether barcodes it uniquely retains are transcriptionally closer to confirmed cells than barcodes it removes, from a raw unfiltered count matrix.

## When NOT to use

- The input is already a filtered cell×gene matrix — the partition requires the raw unfiltered matrix so EmptyNN-removed low-count barcodes are present
- The goal is ambient RNA correction rather than barcode-group comparison — EmptyNN does not correct ambient RNA; tools such as SoupX are needed for that
- There is only a single cell-calling result with no competing filter definition (e.g., no original-study threshold) to form the All-retained contrast group

## Inputs

- Raw unfiltered cell hashing count matrix (h5 or mtx format) from 10X Genomics Cell Ranger
- EmptyNN classification output (nn.keep boolean vector and per-barcode probabilities from emptynn())
- Original study's filtering threshold (200 total counts for the cell hashing dataset)

## Outputs

- de_all_vs_removed.csv — DE table for All-retained vs EmptyNN-removed (adjusted p < 0.01 filter)
- de_all_vs_retained.csv — DE table for All-retained vs EmptyNN-retained (adjusted p < 0.01 filter)
- Downsampled Seurat objects for each barcode group

## How to apply

Load the raw unfiltered cell hashing count matrix (h5 or mtx) and the EmptyNN results from emptynn(counts, threshold = 100, k = 10, iteration = 10) to obtain the nn.keep vector. Partition barcodes into three groups: All-retained (barcodes passing the original study's filtering threshold of 200 total counts, the Cell Ranger v.2-style cutoff), EmptyNN-retained (nn.keep == TRUE), and EmptyNN-removed (nn.keep == FALSE). To remove bias from unbalanced cell numbers and total UMI counts, downsample both counts and cells so groups are balanced between contrasts. Build Seurat objects for the downsampled groups and run FindMarkers() for All-retained vs EmptyNN-removed and All-retained vs EmptyNN-retained, filtering at adjusted p-value < 0.01. Save the two DE tables as de_all_vs_removed.csv and de_all_vs_retained.csv.

## Related tools

- **EmptyNN** (Produces the nn.keep classification used to define EmptyNN-retained and EmptyNN-removed barcode groups from the raw count matrix) — https://github.com/lkmklsmn/empty_nn
- **Seurat** (Builds Seurat objects for downsampled barcode groups and runs FindMarkers() for the two differential expression contrasts)
- **Cell Ranger v.2** (Source of the original study's total-count filtering threshold (200) used to define the All-retained group)

## Examples

```
counts <- t(Read10X_h5("data/neurons_900_raw.h5", use.names=TRUE, unique.features=TRUE)); nn.res <- emptynn(counts, threshold=100, k=10, iteration=10); removed <- counts[!nn.res$nn.keep, ]
```

## Evaluation signals

- All-retained vs EmptyNN-removed yields 3,898 significant DE genes at adjusted p < 0.01
- All-retained vs EmptyNN-retained yields only 155 significant DE genes at adjusted p < 0.01, supporting that EmptyNN-retained barcodes resemble genuine cells
- Group sizes and total UMI counts are balanced across contrasts after downsampling counts and cells
- Barcode partition reproduces reported EmptyNN classification: 20,833 (52.3%) cell-free and 19,009 (47.7%) cell-containing of 39,842 barcodes in the cell hashing dataset
- Both output files de_all_vs_removed.csv and de_all_vs_retained.csv exist with adjusted p-value columns

## Limitations

- EmptyNN does not estimate corrected gene expression profiles; ambient RNA contamination in gene expression estimates requires an external tool such as SoupX, which affects DE comparisons involving EmptyNN-removed barcodes
- The assumption that low total count barcodes are cell-free may not hold when genuine low-count cells exist; the T threshold must be adjusted manually
- Retained cell-containing droplets may have a high fraction of mitochondrial reads and low-quality cells require separate removal in downstream analysis

## Evidence
