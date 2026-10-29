---
name: cell-hashing-data-analysis
description: Use when validating a cell-calling method (e.g., EmptyNN) on a cell hashing
  dataset by asking whether retained barcodes are transcriptionally closer to confirmed
  cells than removed barcodes within the same cell type.
when_to_use_negative:
- Input is already a filtered/cell-called matrix — partitioning into All-retained/EmptyNN-retained/EmptyNN-removed
  requires the raw unfiltered matrix
- Goal is ambient RNA correction of expression profiles — EmptyNN does not estimate
  corrected gene expression; use SoupX instead
- Analysis targets snRNA-seq or other non-cell-hashing datasets without HTO-derived
  labels — validation contrasts should be run on the cell hashing ground truth
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: EmptyNN
  role: Cell-calling via positive-unlabeled learning; its nn.keep vector defines EmptyNN-retained
    and EmptyNN-removed barcode groups used in the DE contrasts
  repo: https://github.com/lkmklsmn/empty_nn
- name: Seurat
  role: Builds downsampled per-group objects and runs FindMarkers() for the two differential
    expression contrasts
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.patter.2021.100311
    title: 'EmptyNN: A neural network based on positive and unlabeled learning to
      remove cell-free droplets and recover lost cells in scRNA-seq data'
  source_content_hash: sha256:49aa7ea61aa8194d86ead70f54d4a6f6c9de660a074daf7948ffc2103fb7b1c8
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
yes_rate: 0.8888888888888888
n_claims: 9
---

# cell-hashing-data-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Analyzes the Stoeckius cell hashing scRNA-seq dataset by partitioning barcodes into All-retained, EmptyNN-retained, and EmptyNN-removed groups and running Seurat differential expression contrasts to test whether EmptyNN-retained barcodes resemble genuine cell-containing barcodes. Validates EmptyNN's cell-calling decisions at the transcriptome level.

## When to use

Use when validating a cell-calling method (e.g., EmptyNN) on a cell hashing dataset by asking whether retained barcodes are transcriptionally closer to confirmed cells than removed barcodes within the same cell type.

## When NOT to use

- Input is already a filtered/cell-called matrix — partitioning into All-retained/EmptyNN-retained/EmptyNN-removed requires the raw unfiltered matrix
- Goal is ambient RNA correction of expression profiles — EmptyNN does not estimate corrected gene expression; use SoupX instead
- Analysis targets snRNA-seq or other non-cell-hashing datasets without HTO-derived labels — validation contrasts should be run on the cell hashing ground truth

## Inputs

- Raw unfiltered cell hashing count matrix (h5 or mtx format, e.g., from Read10X_h5)
- EmptyNN classification results (nn.keep boolean vector and per-barcode probabilities from emptynn())
- Original study filtering threshold (200 total counts) for defining All-retained barcodes

## Outputs

- de_all_vs_removed.csv — significant DE genes (adjusted p < 0.01) for All-retained vs EmptyNN-removed
- de_all_vs_retained.csv — significant DE genes (adjusted p < 0.01) for All-retained vs EmptyNN-retained

## How to apply

Load the raw unfiltered cell hashing count matrix (h5/mtx) together with the EmptyNN classification from emptynn(counts, threshold = 100, k = 10, iteration = 10). Partition barcodes into three groups: All-retained (passing the original Cell Ranger v.2-style threshold of 200 total counts used in the Stoeckius study), EmptyNN-retained (nn.keep == TRUE), and EmptyNN-removed (nn.keep == FALSE). To remove bias from unbalanced cell numbers and total UMI counts, downsample counts and cells so group sizes and UMI totals are balanced between contrasts. Build Seurat objects per group and run FindMarkers() for two contrasts: All-retained vs EmptyNN-removed and All-retained vs EmptyNN-retained. Filter both marker tables at adjusted p-value < 0.01; a much smaller DE gene count for All-retained vs EmptyNN-retained supports that EmptyNN-retained barcodes resemble genuine cells.

## Related tools

- **EmptyNN** (Cell-calling via positive-unlabeled learning; its nn.keep vector defines EmptyNN-retained and EmptyNN-removed barcode groups used in the DE contrasts) — https://github.com/lkmklsmn/empty_nn
- **Seurat** (Builds downsampled per-group objects and runs FindMarkers() for the two differential expression contrasts)

## Examples

```
nn.res <- emptynn(counts, threshold = 100, k = 10, iteration = 10, verbose = TRUE); markers <- FindMarkers(seurat_obj, ident.1 = 'all_retained', ident.2 = 'emptynn_removed')
```

## Evaluation signals

- All-retained vs EmptyNN-removed yields ~3,898 significant DE genes at adjusted p < 0.01
- All-retained vs EmptyNN-retained yields only ~155 significant DE genes at adjusted p < 0.01
- Downsampling leaves group sizes and total UMI counts balanced between the two contrasts
- Both DE tables contain columns for gene names, log fold change, and adjusted p-values, exported as de_all_vs_removed.csv and de_all_vs_retained.csv

## Limitations

- The T threshold (default 100) for defining set P must be chosen manually; the assumption that low-total-count barcodes are cell-free may not hold when genuine low-count cells exist
- EmptyNN-retained droplets may have high mitochondrial read fractions; low-quality cells require separate removal in downstream processing
- EmptyNN does not correct ambient RNA contamination, which can inflate transcriptomic differences between retained and removed barcodes

## Evidence
