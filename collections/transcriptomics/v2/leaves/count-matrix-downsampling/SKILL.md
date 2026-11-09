---
name: count-matrix-downsampling
description: Use when comparing gene expression between barcode groups of unequal
  size or total UMI content (e.g., All-retained vs EmptyNN-removed) so that DE results
  are not biased by depth or cell-number imbalance.
when_to_use_negative:
- Input is a single-cell dataset whose groups are already matched in cell number and
  total UMI counts — downsampling would discard data without benefit
- The analysis goal is cell calling itself rather than fair post-hoc comparison of
  barcode groups — use EmptyNN/EmptyDrops directly instead
- Ambient RNA correction is needed before comparing groups — EmptyNN does not correct
  expression estimates; apply a tool like SoupX first
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: Seurat
  role: Builds Seurat objects from downsampled count matrices and runs FindMarkers()
    for the All-retained vs EmptyNN-removed and All-retained vs EmptyNN-retained contrasts
  repo: https://github.com/lkmklsmn/empty_nn
- name: EmptyNN
  role: Supplies the nn.keep classification that partitions barcodes into EmptyNN-retained
    and EmptyNN-removed groups prior to balanced downsampling
  repo: https://github.com/lkmklsmn/empty_nn
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.patter.2021.100311
    title: 'EmptyNN: A neural network based on positive and unlabeled learning to
      remove cell-free droplets and recover lost cells in scRNA-seq data'
  source_content_hash: sha256:b06cca022aecded18e90ecdf9951b1bb5284d545ff50ba3487fe861627272b6f
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
  edam_operation: http://edamontology.org/operation_3221
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

# Count-matrix downsampling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Balances UMI counts and cell numbers across barcode groups (e.g., All-retained, EmptyNN-retained, EmptyNN-removed) by downsampling counts and cells before differential expression, removing confounding from unbalanced group sizes in the EmptyNN cell hashing analysis.

## When to use

Use when comparing gene expression between barcode groups of unequal size or total UMI content (e.g., All-retained vs EmptyNN-removed) so that DE results are not biased by depth or cell-number imbalance.

## When NOT to use

- Input is a single-cell dataset whose groups are already matched in cell number and total UMI counts — downsampling would discard data without benefit
- The analysis goal is cell calling itself rather than fair post-hoc comparison of barcode groups — use EmptyNN/EmptyDrops directly instead
- Ambient RNA correction is needed before comparing groups — EmptyNN does not correct expression estimates; apply a tool like SoupX first

## Inputs

- Raw unfiltered cell hashing count matrix (h5 or mtx format) with barcodes as rows/columns
- EmptyNN classification results (nn.keep boolean vector from emptynn())
- Original study filtering threshold (200 total counts) defining the All-retained group

## Outputs

- Downsampled, depth- and size-balanced count matrices per barcode group
- Differential expression tables (e.g., de_all_vs_removed.csv, de_all_vs_retained.csv) filtered at adjusted p < 0.01
- Counts of significant DE genes per contrast for comparison across contrasts

## How to apply

Partition barcodes into groups — All-retained (barcodes passing the original study's 200-total-count threshold), EmptyNN-retained (nn.keep from emptynn(counts, threshold = 100, k = 10, iteration = 10)), and EmptyNN-removed. Because the groups differ substantially in both cell count and sequencing depth, downsample UMI counts and cells so that group sizes and total UMI counts are balanced between contrasts; the paper does this explicitly 'to remove any bias from unbalanced numbers of cells in either group, as well as total UMI count'. Build Seurat objects from the downsampled matrices and run FindMarkers() for All-retained vs EmptyNN-removed and All-retained vs EmptyNN-retained. Filter markers at adjusted p < 0.01 and compare the number of significant genes between contrasts to judge whether one group resembles genuine cells more closely.

## Related tools

- **Seurat** (Builds Seurat objects from downsampled count matrices and runs FindMarkers() for the All-retained vs EmptyNN-removed and All-retained vs EmptyNN-retained contrasts) — https://github.com/lkmklsmn/empty_nn
- **EmptyNN** (Supplies the nn.keep classification that partitions barcodes into EmptyNN-retained and EmptyNN-removed groups prior to balanced downsampling) — https://github.com/lkmklsmn/empty_nn

## Examples

```
nn.res <- emptynn(counts, threshold = 100, k = 10, iteration = 10); FindMarkers(seurat_obj, ident.1 = 'All-retained', ident.2 = 'EmptyNN-removed')
```

## Evaluation signals

- After downsampling, total UMI counts and cell numbers are balanced across the groups being contrasted
- All-retained vs EmptyNN-removed yields 3,898 significant DE genes at adjusted p < 0.01 in CD4 T cells
- All-retained vs EmptyNN-retained yields only 155 significant DE genes at adjusted p < 0.01 in CD4 T cells
- Much smaller DE gene count for All-retained vs EmptyNN-retained than vs EmptyNN-removed, supporting that EmptyNN-retained barcodes resemble genuine cells
- DE tables are saved with the expected schema as de_all_vs_removed.csv and de_all_vs_retained.csv

## Limitations

- Downsampling discards data; results depend on the random subsampling of cells and UMI counts and should be checked for reproducibility
- Balancing groups cannot remove systematic biological differences, only depth and size confounding
- EmptyNN does not correct ambient RNA contamination in expression estimates, which may affect DE comparisons; SoupX or similar correction may be needed

## Evidence

- [results] Differential expression findings in CD4 T cells: [evidence span withheld — anchor status: not_found]
