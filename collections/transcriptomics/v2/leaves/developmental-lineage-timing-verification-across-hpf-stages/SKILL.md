---
name: developmental-lineage-timing-verification-across-hpf-stages
description: Use when integrating time-course embryo scRNA-seq datasets (e.g., zebrafish
  Wagner et al. and Farrell et al.) to verify or compare the developmental time point
  at which major lineages first emerge.
when_to_use_negative:
- The reference data does not contain the lineages of interest — MASI alone cannot
  annotate unseen cell types not present in the reference; combine with SCCAF instead
- The reference dataset is low quality or mismatched in resolution to the query —
  annotation accuracy depends critically on reference choice and resolution
- The goal is batch-effect correction for visualization rather than marker-based label
  transfer — model-based integration methods (Seurat, Harmony, LIGER) would be the
  wrong fit for MASI's model-free workflow, and vice versa this skill is not a substitute
  for those
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: MASI
  role: 'Core method: identifies markers from the end-point reference, converts gene
    expression to cell-type scores (gene2cell), and transfers labels (parallel/MACA
    ensemble)'
  repo: https://github.com/hayatlab/MASI
- name: MACA
  role: Over-clusters cells at multiple resolutions (3, 5, 7) and neighborhood sizes
    (5, 10, 15) to map ensemble lineage labels onto query data
- name: Scanpy
  role: Performs LogNormalization of raw counts (scale factor 10,000, log-transform)
    and handles Anndata objects
- name: SCCAF
  role: Optional companion to identify subtypes when query data contains cell types
    not present in the reference
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-023-04820-3
    title: MASI enables fast model-free standardization and integration of single-cell
      transcriptomics data
  source_content_hash: sha256:7d5ddbdab3d3523f9bde606913d5595f93a84fc5627e5edb1e13a350e0452ab9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hayatlab/MASI
  tools:
  - MACA
  - MASI
  - SCCAF
  - Scanpy
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# developmental-lineage-timing-verification-across-hpf-stages

## Summary

Reproduces developmental lineage-emergence timings by integrating embryo scRNA-seq time-course datasets with MASI's marker-based cell-type score matrix, then tracking when each lineage (germline, epidermis, notochord, non-neural ectoderm) first appears across hpf time points.

## When to use

Use when integrating time-course embryo scRNA-seq datasets (e.g., zebrafish Wagner et al. and Farrell et al.) to verify or compare the developmental time point at which major lineages first emerge.

## When NOT to use

- The reference data does not contain the lineages of interest — MASI alone cannot annotate unseen cell types not present in the reference; combine with SCCAF instead
- The reference dataset is low quality or mismatched in resolution to the query — annotation accuracy depends critically on reference choice and resolution
- The goal is batch-effect correction for visualization rather than marker-based label transfer — model-based integration methods (Seurat, Harmony, LIGER) would be the wrong fit for MASI's model-free workflow, and vice versa this skill is not a substitute for those

## Inputs

- Processed zebrafish embryo scRNA-seq datasets (Wagner et al. and Farrell et al.) with developmental time-point metadata
- End-point reference dataset with cell-type labels ('cell_type') in Anndata format
- Cell-type marker dictionary (cell_markers) from the MASI figshare deposit 18866264

## Outputs

- zebrafish_lineage_annotations.csv with per-cell lineage labels and time-point assignments
- Cell-type score matrix usable for visualization and downstream analyses
- Lineage-emergence timing calls per hpf stage (e.g., germline at 3 hpf, notochord ~7 hpf)

## How to apply

Download the processed Wagner et al. and Farrell et al. zebrafish embryo datasets plus the end-point reference marker list from the MASI figshare deposit (18866264). Preprocess raw counts by LogNormalization (divide by total counts per cell, scale factor 10,000, log-transform) with Scanpy, then convert the combined gene expression matrix to a cell-type score matrix via masi.gene2cell with linear marker weighting (1st marker 100%, Nth marker 50%) and the 0.25 percentile PlinerScore threshold. Annotate cells with masi.parallel(scores, labels, batch_size=50000), which over-clusters with MACA across resolutions 3, 5, 7 and neighborhood sizes 5, 10, 15, one batch per CPU core, and ensembles labels into the 8 major lineages. Track each lineage's presence across developmental time points (e.g., germline and epidermis at 3 hpf, notochord ~7 hpf, non-neural ectoderm separation at 12 hpf) and export annotated cells with lineage labels and time-point assignments to zebrafish_lineage_annotations.csv.

## Related tools

- **MASI** (Core method: identifies markers from the end-point reference, converts gene expression to cell-type scores (gene2cell), and transfers labels (parallel/MACA ensemble)) — https://github.com/hayatlab/MASI
- **MACA** (Over-clusters cells at multiple resolutions (3, 5, 7) and neighborhood sizes (5, 10, 15) to map ensemble lineage labels onto query data)
- **Scanpy** (Performs LogNormalization of raw counts (scale factor 10,000, log-transform) and handles Anndata objects)
- **SCCAF** (Optional companion to identify subtypes when query data contains cell types not present in the reference)

## Examples

```
import MASI as masi; scores, labels = masi.gene2cell(ad=ad, cell_markers=cell_markers, use_weight=True); annotation = masi.parallel(scores=scores, labels=labels, batch_size=50000)
```

## Evaluation signals

- Germline lineage is retrieved back to at least the 3 hpf time point, consistent with Farrell et al.'s report that the germ layer appears before 4 hpf
- All four lineage-timing calls match published Fig. 1b–d annotations: germline at 3 hpf, epidermis at 3 hpf, notochord around 7 hpf, non-neural ectoderm separation at 12 hpf
- Cell-type score matrix is produced with use_weight=True linear weighting (100% for 1st marker down to 50% for Nth) and the 0.25 percentile PlinerScore threshold applied
- MACA ensemble spans all parameter combinations: clustering resolutions 3, 5, 7 with neighborhood sizes 5, 10, 15, batched at 50,000 cells per CPU core
- Output CSV assigns every cell exactly one of the 8 ensembled major lineage labels plus a time-point assignment

## Limitations

- Annotation accuracy depends on the quality of the reference data; the choice and resolution of the reference are critical to downstream analysis
- MASI alone cannot annotate unseen cell types not present in the reference — SCCAF must be combined to identify subtypes within major cell types
- No software version numbers are reported for tools used (Scanpy, MASI, MACA, etc.), complicating exact reproduction
- The number of batches N for splitting large-scale data is unspecified; it depends on available CPU cores

## Evidence

- [results] Lineage timing from integrated map: [evidence span withheld — anchor status: not_found]
- [methods] gene2cell invocation: "scores, labels = masi.gene2cell(ad=ad,cell_markers=cell_markers,use_weight=True)"
- [methods] parallel invocation: "annotation= masi.parallel(scores=scores,labels=labels,batch_size=50000)"
- [discussion] PlinerScore threshold: "The default value for PlinerScore threshold is 0.25 as the percentile threshold"
- [discussion] MACA multi-parameter over-clustering: "These include clustering resolution 3, 5, 7 with 5, 10, 15 as neighborhood sizes to over-cluster cells"
- [discussion] Marker linear weighting: "the 1st marker in this ranked list will contribute 100% of its expression to the cell-type score of A, while the Nth marker only contributes 50%"
- [discussion] Reference quality dependency: "accurate annotation via MASI is also dependent on the quality of reference data."
- [readme] MASI install requirements including lineage-analysis extras: [evidence span withheld — anchor status: not_found]
