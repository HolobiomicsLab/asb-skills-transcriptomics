---
name: over-clustering-ensemble-label-transfer
description: Use when transferring reference cell-type labels onto a large query single-cell
  dataset (AnnData) via marker-based cell-type scores, and you need robust consensus
  annotations by over-clustering across multiple MACA parameter settings rather than
  a single clustering run.
when_to_use_negative:
- Query data contains cell types or subtypes absent from the reference — MASI alone
  cannot annotate unseen cell types; combine MASI with SCCAF to identify subtypes
  within major cell types.
- The input has not been converted to a cell-type score matrix (masi.gene2cell not
  yet run) — this skill operates on scores, not raw gene expression.
- Annotation accuracy is critical and the reference data quality/resolution is uncertain
  — annotation via MASI depends on reference quality, and the choice and resolution
  of the reference are critical to downstream analysis.
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MASI
  role: Provides the gene2cell score conversion and masi.parallel API that runs the
    MACA over-clustering parameter grid and ensembles labels
  repo: https://github.com/hayatlab/MASI
- name: MACA
  role: Over-clustering engine that maps cell-type labels to query data at resolutions
    3, 5, 7 and neighborhood sizes 5, 10, 15, one batch per CPU core
- name: Scanpy
  role: Preprocessing of raw counts by LogNormalization (divide by total counts, scale
    factor 10,000, log-transform) before score conversion
- name: SCCAF
  role: Optional companion to identify subtypes within major cell types when unseen
    cell types are not in the reference
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-023-04820-3
    title: MASI enables fast model-free standardization and integration of single-cell
      transcriptomics data
  source_content_hash: sha256:e88f953a45b6232a82a824397602b02d0253f3fefb6b980cf986379c8ec25230
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hayatlab/MASI
  tools:
  - MACA
  - MASI
  - SCCAF
  - Scanpy
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# Over-clustering ensemble label transfer

## Summary

A MASI-based label-transfer strategy that deliberately over-clusters cells with MACA across multiple clustering resolutions (3, 5, 7) and neighborhood sizes (5, 10, 15), then ensembles the resulting labels into stable major cell-type/lineage annotations, with large datasets split into N batches (default batch_size=50000) processed one batch per CPU core.

## When to use

Use when transferring reference cell-type labels onto a large query single-cell dataset (AnnData) via marker-based cell-type scores, and you need robust consensus annotations by over-clustering across multiple MACA parameter settings rather than a single clustering run.

## When NOT to use

- Query data contains cell types or subtypes absent from the reference — MASI alone cannot annotate unseen cell types; combine MASI with SCCAF to identify subtypes within major cell types.
- The input has not been converted to a cell-type score matrix (masi.gene2cell not yet run) — this skill operates on scores, not raw gene expression.
- Annotation accuracy is critical and the reference data quality/resolution is uncertain — annotation via MASI depends on reference quality, and the choice and resolution of the reference are critical to downstream analysis.

## Inputs

- AnnData object (ad) containing log-normalized counts (scale factor 10,000, log-transformed via Scanpy) for the combined reference and query datasets
- cell_markers: dict of cell-type marker genes identified from the reference data (e.g., via masi.marker_identification with robust rank aggregation)
- Reference labels array aligned to cell types in the marker dict

## Outputs

- scores: cell-type score matrix (genes x cells converted to cell-type scores) usable for visualization and downstream analyses
- labels: cell-type labels assigned per cell after MACA over-clustering and ensembling (e.g., the 8 major zebrafish lineages)
- Annotated cell table (e.g., zebrafish_lineage_annotations.csv with lineage labels and time-point assignments)

## How to apply

First convert the combined gene expression matrix to a cell-type score matrix with masi.gene2cell(ad=ad, cell_markers=cell_markers, use_weight=True), applying linear marker weighting (1st marker contributes 100%, Nth marker 50%) and suppressing marker expression below the 0.25 percentile PlinerScore threshold. Then run masi.parallel(scores=scores, labels=labels, batch_size=50000), which maps labels to query data using MACA with over-clustering at clustering resolutions 3, 5, and 7 and neighborhood sizes 5, 10, and 15, splitting the data into N batches with one MACA run per CPU core. Finally ensemble the labels produced across the parameter grid into the target set of major cell types (e.g., the 8 zebrafish embryonic lineages). Over-clustering is intentional: varying resolution and neighborhood size lets the ensemble recover consistent major lineages despite clustering instability at any single setting. No highly variable gene selection is needed because only the identified marker genes are used for integrative annotation.

## Related tools

- **MASI** (Provides the gene2cell score conversion and masi.parallel API that runs the MACA over-clustering parameter grid and ensembles labels) — https://github.com/hayatlab/MASI
- **MACA** (Over-clustering engine that maps cell-type labels to query data at resolutions 3, 5, 7 and neighborhood sizes 5, 10, 15, one batch per CPU core)
- **Scanpy** (Preprocessing of raw counts by LogNormalization (divide by total counts, scale factor 10,000, log-transform) before score conversion)
- **SCCAF** (Optional companion to identify subtypes within major cell types when unseen cell types are not in the reference)

## Examples

```
scores, labels = masi.gene2cell(ad=ad, cell_markers=cell_markers, use_weight=True); annotation = masi.parallel(scores=scores, labels=labels, batch_size=50000)
```

## Evaluation signals

- Ensemble labels collapse into the expected set of major lineages/cell types (e.g., 8 major zebrafish embryonic lineages).
- Lineage-timing calls match published annotations: germline present at 3 hpf, epidermis at 3 hpf, notochord emerging around 7 hpf, non-neural ectoderm separation at 12 hpf (Fig. 1b–d).
- The over-clustering grid covers all nine MACA settings (resolutions 3, 5, 7 x neighborhood sizes 5, 10, 15) before ensembling.
- Large-data runs are split into batches of ~50000 cells with one MACA run per CPU core, confirming the parallel execution path was used.
- Marker expression values below the 0.25 percentile PlinerScore threshold were suppressed to zero prior to score computation.

## Limitations

- Accurate annotation via MASI is dependent on the quality of reference data; the choice and resolution of the reference are critical to downstream analysis.
- If query data has unseen cell types not in the reference, MASI alone cannot annotate them; MASI in combination with SCCAF is required to identify subtypes within major cell types.
- The number of batches N used to split large-scale data is unspecified, and the PlinerScore threshold is described with both a variable 'X-percentile' and a default of 0.25 — the applied value per analysis is not always stated.
- No software version numbers are reported for the tools involved (Scanpy, scikit-learn, MASI, MACA), complicating exact reproduction.

## Evidence

- [discussion] Label transferring through MASI using MACA with multiple clustering parameters: "These include clustering resolution 3, 5, 7 with 5, 10, 15 as neighborhood sizes to over-cluster cells"
- [discussion] Large-scale data split into N batches, one MACA run per CPU core: "we split the whole data into N batches and ran MACA with one batch per CPU core"
- [discussion] MACA performs the label mapping to query data: "Mapping cell-type labels to query data is performed using MACA"
- [methods] Concrete masi.parallel invocation with batch_size 50000: "annotation= masi.parallel(scores=scores,labels=labels,batch_size=50000)"
- [readme] README: MASI transfers labels to target data through MACA; sccaf optional: "MASI utilizes robust marker idenfication to identify marker genes from reference data and transfers cell-type labels to target data through MACA."
