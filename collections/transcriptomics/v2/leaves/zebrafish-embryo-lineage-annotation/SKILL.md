---
name: zebrafish-embryo-lineage-annotation
description: Use when integrating multiple zebrafish embryo scRNA-seq time-course
  datasets (e.g., Wagner et al. and Farrell et al.) against an end-point reference
  marker list to recover lineage emergence timings across developmental stages.
when_to_use_negative:
- The query data contains cell types not present in the end-point reference — MASI
  alone cannot annotate unseen cell types and would need to be combined with SCCAF
  to identify subtypes.
- The goal is batch-corrected latent-space integration (e.g., Seurat, Harmony, LIGER)
  rather than fast marker-based label transfer — MASI deliberately skips HVG selection
  and model training.
- No high-quality reference with cell-type markers at matching resolution is available,
  since MASI annotation accuracy depends on reference quality.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0654
tools:
- name: MASI
  role: 'Marker-assisted standardization and integration: gene2cell converts expression
    to cell-type scores; parallel transfers labels across batches'
  repo: https://github.com/hayatlab/MASI
- name: MACA
  role: Over-clustering engine used by MASI to map cell-type labels to query data
    at clustering resolutions 3, 5, 7 and neighborhood sizes 5, 10, 15
- name: Scanpy
  role: 'Preprocessing: LogNormalization of raw counts (scale factor 10,000, log-transform);
    optional fa2 install for integrative lineage analysis'
- name: SCCAF
  role: Optional companion for identifying subtypes when the query has unseen cell
    types not in the reference
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-023-04820-3
    title: MASI enables fast model-free standardization and integration of single-cell
      transcriptomics data
  source_content_hash: sha256:cd9e0f8da9004c873f29e57ff0386c6d3791f730306b74676bad534062348f76
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
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# zebrafish-embryo-lineage-annotation

## Summary

Reproduce zebrafish embryo lineage-emergence timings by integrating Wagner et al. and Farrell et al. datasets with MASI's marker-based cell-type scoring and MACA label transfer, using end-point zebrafish data as the reference. Demonstrates that a model-free marker approach recovers germline at 3 hpf, epidermis at 3 hpf, notochord around 7 hpf, and non-neural ectoderm separation at 12 hpf (Fig. 1b–d).

## When to use

Use when integrating multiple zebrafish embryo scRNA-seq time-course datasets (e.g., Wagner et al. and Farrell et al.) against an end-point reference marker list to recover lineage emergence timings across developmental stages.

## When NOT to use

- The query data contains cell types not present in the end-point reference — MASI alone cannot annotate unseen cell types and would need to be combined with SCCAF to identify subtypes.
- The goal is batch-corrected latent-space integration (e.g., Seurat, Harmony, LIGER) rather than fast marker-based label transfer — MASI deliberately skips HVG selection and model training.
- No high-quality reference with cell-type markers at matching resolution is available, since MASI annotation accuracy depends on reference quality.

## Inputs

- Processed Wagner et al. zebrafish embryo scRNA-seq dataset (from MASI figshare deposit 18866264)
- Processed Farrell et al. zebrafish embryo scRNA-seq dataset (from MASI figshare deposit 18866264)
- End-point zebrafish reference dataset with cell-type labels ('cell_type') for marker identification
- Cell-type marker dictionary (cell_markers) from the end-point reference

## Outputs

- Cell-type score matrix (scores) and per-cell-type labels from masi.gene2cell
- Ensembled lineage annotations for all cells (8 major lineages) via masi.parallel
- zebrafish_lineage_annotations.csv with per-cell lineage labels and time-point assignments
- Lineage-emergence time calls: germline at 3 hpf, epidermis at 3 hpf, notochord around 7 hpf, non-neural ectoderm separation at 12 hpf (reproducing Fig. 1b–d)

## How to apply

Download the processed Wagner et al. and Farrell et al. datasets and the end-point reference marker list from the MASI figshare deposit (18866264). Preprocess raw counts with LogNormalization (divide by total counts per cell, scale factor 10,000, log-transform) using Scanpy; note MASI skips highly variable gene selection because only marker genes are used. Convert the gene expression matrix to a cell-type score matrix with masi.gene2cell(ad=ad, cell_markers=cell_markers, use_weight=True), applying the linear marker weighting (1st marker 100%, Nth marker 50%) and the 0.25 percentile PlinerScore threshold that suppresses below-threshold marker expression to zero. Annotate cells with masi.parallel(scores=scores, labels=labels, batch_size=50000), which over-clusters with MACA across resolutions 3, 5, 7 and neighborhood sizes 5, 10, 15, one batch per CPU core, and ensembles labels into the 8 major lineages. Finally, track each lineage's first appearance across time points (3, 4, 7, 12 hpf) and write zebrafish_lineage_annotations.csv.

## Related tools

- **MASI** (Marker-assisted standardization and integration: gene2cell converts expression to cell-type scores; parallel transfers labels across batches) — https://github.com/hayatlab/MASI
- **MACA** (Over-clustering engine used by MASI to map cell-type labels to query data at clustering resolutions 3, 5, 7 and neighborhood sizes 5, 10, 15)
- **Scanpy** (Preprocessing: LogNormalization of raw counts (scale factor 10,000, log-transform); optional fa2 install for integrative lineage analysis)
- **SCCAF** (Optional companion for identifying subtypes when the query has unseen cell types not in the reference)

## Examples

```
import MASI as masi; scores, labels = masi.gene2cell(ad=ad, cell_markers=cell_markers, use_weight=True); annotation = masi.parallel(scores=scores, labels=labels, batch_size=50000)
```

## Evaluation signals

- Lineage-timing calls match published Fig. 1b–d: germline present at 3 hpf (not first at 4 hpf as in Wagner et al.), epidermal lineage at 3 hpf, notochord emerging around 7 hpf, non-neural ectoderm separation at 12 hpf
- Germline development is retrieved back to at least the 3 hpf time point, consistent with Farrell et al.'s report that the germ layer appears before 4 hpf
- zebrafish_lineage_annotations.csv is produced with per-cell lineage labels and time-point assignments across all cells of both datasets
- masi.parallel output ensembles MACA annotations across all 9 parameter combinations (resolutions 3/5/7 × neighborhood sizes 5/10/15) into the 8 major lineages
- PlinerScore conversion used the 0.25 percentile threshold with below-threshold marker expression suppressed to zero

## Limitations

- MASI annotation accuracy depends on the quality of reference data; the choice and resolution of the reference are critical to downstream analysis
- MASI alone cannot annotate unseen cell types not present in the reference; combine with SCCAF to identify subtypes within major cell types
- The applied PlinerScore threshold is described with both a variable 'X-percentile' and a default of 0.25; the per-analysis value is not explicitly stated in the text
- Number of batches N for splitting large-scale data (one MACA run per CPU core) is unspecified; batch_size=50000 is used in the tutorial

## Evidence

- [methods] gene2cell converts expression to scores: "scores, labels = masi.gene2cell(ad=ad,cell_markers=cell_markers,use_weight=True)"
- [methods] parallel annotates via MACA: "annotation= masi.parallel(scores=scores,labels=labels,batch_size=50000)"
- [discussion] LogNormalization preprocessing: "Raw gene expression counts data were ‘LogNormalized’, which divides the total count in that cell and multiplies it by a scale factor of 10,000"
- [discussion] PlinerScore default threshold: "The default value for PlinerScore threshold is 0.25 as the percentile threshold"
- [discussion] MASI over-clustering parameters: "These include clustering resolution 3, 5, 7 with 5, 10, 15 as neighborhood sizes to over-cluster cells"
- [readme] MASI approach overview: "MASI utilizes robust marker idenfication to identify marker genes from reference data and transfers cell-type labels to target data through MACA."
