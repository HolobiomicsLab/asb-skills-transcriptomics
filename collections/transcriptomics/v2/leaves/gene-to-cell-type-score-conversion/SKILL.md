---
name: gene-to-cell-type-score-conversion
description: Use when you have a ranked per-cell-type marker list from reference data
  and a LogNormalized query/reference expression matrix (AnnData) that must be converted
  into a per-cell cell-type score matrix for label transfer or integration.
when_to_use_negative:
- When the input is already a cell-type score matrix or a pre-computed feature embedding
  rather than a gene expression matrix plus marker list
- When training large model-based annotation methods (scNym, scArches, Symphony) is
  desired — MASI's marker-based score conversion is a distinct model-free approach
  and skipping HVG selection is incompatible with methods that need variable genes
- When the query contains cell types absent from the reference — scores/labels alone
  will mis-assign them; pair with SCCAF and the certainty-score filter instead
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: MASI
  role: Provides gene2cell (gene-to-cell-type score conversion with use_weight weighting),
    marker_identification, and the downstream parallel annotation step
  repo: https://github.com/hayatlab/MASI
- name: Scanpy
  role: AnnData container handling and pp.highly_variable_genes (explicitly skipped
    for MASI since only marker genes are used)
- name: Seurat
  role: R dependency installed for MASI's marker identification/rank aggregation workflow
provenance:
  source_task_ids:
  - task_008
  - task_006
  source_papers:
  - doi: 10.1038/s42003-023-04820-3
    title: MASI enables fast model-free standardization and integration of single-cell
      transcriptomics data
  source_content_hash: sha256:b04cb87bb932203b4257bd1ec48ca93453db80395fe4fd5160a6af7b58305e6d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hayatlab/MASI
  tools:
  - MASI
  - Scanpy
  - Seurat
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# gene-to-cell-type-score-conversion

## Summary

MASI's gene2cell step transforms a LogNormalized single-cell gene expression AnnData matrix into a cell-type score matrix by summing ranked, linearly discounted marker-gene expression per cell type, with optional PlinerScore thresholding; this score matrix is the critical input for MASI's downstream annotation and integration.

## When to use

Use when you have a ranked per-cell-type marker list from reference data and a LogNormalized query/reference expression matrix (AnnData) that must be converted into a per-cell cell-type score matrix for label transfer or integration.

## When NOT to use

- When the input is already a cell-type score matrix or a pre-computed feature embedding rather than a gene expression matrix plus marker list
- When training large model-based annotation methods (scNym, scArches, Symphony) is desired — MASI's marker-based score conversion is a distinct model-free approach and skipping HVG selection is incompatible with methods that need variable genes
- When the query contains cell types absent from the reference — scores/labels alone will mis-assign them; pair with SCCAF and the certainty-score filter instead

## Inputs

- Combined query/reference gene expression matrix as an AnnData object, LogNormalized (divide by total count, multiply by 10,000, log-transform)
- Ranked cell-type marker dictionary (cell_markers) from reference data, e.g. from masi.marker_identification using robust rank aggregation of top 20 markers per DE test
- Optional use_weight flag enabling the linear 100%-to-50% marker discounting scheme

## Outputs

- Cell-type score matrix (per-cell scores across cell types) usable for visualization and downstream annotation/integration
- Per-cell assigned labels returned by masi.gene2cell (scores, labels tuple)

## How to apply

Identify cell-type markers from the reference AnnData (labels in the 'cell_type' column) via masi.marker_identification, then call masi.gene2cell(ad=ad, cell_markers=cell_markers, use_weight=True) to convert the gene expression matrix into a cell-type score matrix. With use_weight=True, ranked markers are linearly discounted so the 1st marker contributes 100% of its expression to the cell-type score and the Nth marker only 50%, with intermediate markers discounted linearly. Before score conversion, the pipeline applies a PlinerScore transformation: suppress marker expression values to zero if they fall below the percentile threshold (default 0.25 percentile across all cells) after TF-IDF weighting. Highly variable gene selection is deliberately skipped because only the identified marker genes are used for integrative annotation. The returned scores and labels feed downstream steps such as masi.parallel annotation, MACA-based label transfer, and certainty-score (1 - D1st/DNth) computation for unassigned-cell detection.

## Related tools

- **MASI** (Provides gene2cell (gene-to-cell-type score conversion with use_weight weighting), marker_identification, and the downstream parallel annotation step) — https://github.com/hayatlab/MASI
- **Scanpy** (AnnData container handling and pp.highly_variable_genes (explicitly skipped for MASI since only marker genes are used))
- **Seurat** (R dependency installed for MASI's marker identification/rank aggregation workflow)

## Examples

```
import MASI as masi; scores, labels = masi.gene2cell(ad=ad, cell_markers=cell_markers, use_weight=True)
```

## Evaluation signals

- gene2cell returns exactly a (scores, labels) pair whose scores matrix has cells as rows and reference cell types as columns
- With use_weight=True, weighted marker contributions fall monotonically from 1.0 (rank 1) to 0.5 (rank N) and all weights lie in [0.5, 1.0]
- Marker expression values below the PlinerScore percentile threshold (default 0.25 across all cells) are suppressed to zero before summing marker expression per cell type
- Highly variable gene selection was skipped; the feature space equals the identified marker gene set
- Downstream sanity check: masi.parallel(scores, labels, batch_size=50000) yields annotations consistent with the cell-type score labels

## Limitations

- MASI annotation accuracy depends on reference data quality; choice and resolution of the reference are critical to downstream analysis
- MASI alone cannot annotate unseen cell types not present in the reference; combining MASI with SCCAF is needed to identify subtypes within major cell types
- The PlinerScore threshold is described both as a variable 'X-percentile' and a default of 0.25; the applied value per analysis is not stated
- Adding BatchNorm1D as data transformation can remove batch effects but may sacrifice discriminative power for cell-type identification (optional PyTorch step in MASI)

## Evidence

- [methods] scores, labels = masi.gene2cell(ad=ad,cell_markers=cell_markers,use_weight=True): "scores, labels = masi.gene2cell(ad=ad,cell_markers=cell_markers,use_weight=True)"
- [discussion] the 1st marker in this ranked list will contribute 100% of its expression to the cell-type score of A, while the Nth marker only contributes 50%: "the 1st marker in this ranked list will contribute 100% of its expression to the cell-type score of A, while the Nth marker only contributes 50%"
- [discussion] The default value for PlinerScore threshold is 0.25 as the percentile threshold: "The default value for PlinerScore threshold is 0.25 as the percentile threshold"
- [discussion] we skipped the step of calling highly variable genes, because only the identified marker genes were used for integrative annotation.: "we skipped the step of calling highly variable genes, because only the identified marker genes were used for integrative annotation."
- [readme] step 1 transform gene expression matrix to cell-type score matrix: "##step 1 transform gene expression matrix to cell-type score matrix"
