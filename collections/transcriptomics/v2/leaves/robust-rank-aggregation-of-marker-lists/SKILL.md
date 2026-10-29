---
name: robust-rank-aggregation-of-marker-lists
description: Use when building a final ranked marker list per cell type from reference
  scRNA-seq data where individual DE tests vary in their ability to predict cell types
  and a single consensus ranking is needed.
when_to_use_negative:
- The reference AnnData lacks cell-type labels ('cell_type'), since per-cluster DE
  ranking cannot be computed
- The query data contains cell types unseen in the reference — MASI (and thus its
  aggregated markers) cannot annotate unseen types and must be combined with SCCAF
  for subtype discovery
- You already have a validated, curated marker gene list per cell type and do not
  need to re-derive rankings from DE tests
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MASI
  role: Provides marker_identification, marker_identification_fast, and marker_identification_LC
    functions that run the per-cell-type DE tests and perform ensemble rank aggregation
    (RRA) / Lancaster combination
  repo: https://github.com/hayatlab/MASI
- name: RobustRankAggreg
  role: R package implementing robust rank aggregation across the per-DE-test ranked
    marker lists; installed separately in R and called from MASI via rpy2
- name: Seurat
  role: R package installed alongside RobustRankAggreg as part of MASI's marker identification
    dependency stack
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42003-023-04820-3
    title: MASI enables fast model-free standardization and integration of single-cell
      transcriptomics data
  source_content_hash: sha256:0e83a506cf7800c6b8a8af7416ccc387bab9c15727bc47628ecbedc26d8466c3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hayatlab/MASI
  tools:
  - MASI
  - RobustRankAggreg
  - Seurat
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# robust-rank-aggregation-of-marker-lists

## Summary

Constructs a consensus ranked cell-type marker list by aggregating the outputs of multiple differential expression tests with robust rank aggregation (RRA), with an alternative Lancaster p-value combination (marker_identification_LC), as used in MASI to build reference markers for label transfer.

## When to use

Use when building a final ranked marker list per cell type from reference scRNA-seq data where individual DE tests vary in their ability to predict cell types and a single consensus ranking is needed.

## When NOT to use

- The reference AnnData lacks cell-type labels ('cell_type'), since per-cluster DE ranking cannot be computed
- The query data contains cell types unseen in the reference — MASI (and thus its aggregated markers) cannot annotate unseen types and must be combined with SCCAF for subtype discovery
- You already have a validated, curated marker gene list per cell type and do not need to re-derive rankings from DE tests

## Inputs

- LogNormalized reference single-cell expression data in AnnData format (scale factor 10,000 + log1p), with cell-type labels in obs['cell_type']
- Per-DE-test ranked gene lists (top 20 markers per DE test compiled per cell type)
- Optional: per-marker significance p-values from each DE test for Lancaster combination

## Outputs

- cell_markers.csv — a per-cell-type ranked marker table in which each cell type has markers ordered by consensus rank
- cell_markers dict keyed by cell type, consumed by masi.gene2cell(ad=ad, cell_markers=cell_markers, use_weight=True)

## How to apply

Load the processed reference single-cell dataset (e.g. from the MASI Figshare deposit) with raw counts LogNormalized at scale factor 10,000 followed by log-transformation, and cell-type labels stored in AnnData obs as 'cell_type'. Run per-cluster differential expression with MASI's marker_identification or marker_identification_fast to rank genes for each cell type. Compile the top 20 marker genes from each DE test into a pooled candidate set, then build the consensus ranking via robust rank aggregation across the per-test ranked lists (MASI's README requires installing Seurat and RobustRankAggreg separately in R, accessed via rpy2). Independently, aggregate statistically significant markers using Lancaster's method for combining p-values via marker_identification_LC. Write the final ranked marker table (markers ordered per cell type) to cell_markers.csv, which feeds masi.gene2cell for cell-type scoring.

## Related tools

- **MASI** (Provides marker_identification, marker_identification_fast, and marker_identification_LC functions that run the per-cell-type DE tests and perform ensemble rank aggregation (RRA) / Lancaster combination) — https://github.com/hayatlab/MASI
- **RobustRankAggreg** (R package implementing robust rank aggregation across the per-DE-test ranked marker lists; installed separately in R and called from MASI via rpy2)
- **Seurat** (R package installed alongside RobustRankAggreg as part of MASI's marker identification dependency stack)

## Examples

```
cell_markers = masi.marker_identification(source_data=source)
```

## Evaluation signals

- cell_markers.csv exists and contains a ranked marker list for every cell type in the reference obs['cell_type'] column
- Within each cell type, markers are ordered by consensus rank and are consistent with the top-20-per-DE-test compilation described in the paper
- The pooled candidate set per cell type does not exceed the union of the top 20 markers from each DE test
- The Seurat and RobustRankAggreg R packages installed successfully (per the MASI README install requirements: scanpy, cosg, rpy2, plus Seurat and RobustRankAggreg in R)
- Downstream check: scores, labels = masi.gene2cell(ad=ad, cell_markers=cell_markers, use_weight=True) runs and produces a cell-type score matrix from the aggregated markers

## Limitations

- MASI annotation accuracy depends on reference data quality; the choice and resolution of the reference are critical to downstream analysis
- Aggregated markers can only reflect cell types present in the reference; unseen cell types are not handled without combining MASI with SCCAF
- No software version numbers are reported for the tools in the pipeline (Scanpy, MASI, Seurat, RobustRankAggreg), so exact reproducibility of the ranking is not guaranteed

## Evidence

- [discussion] the top 20 marker genes from each DE test were compiled together: "the top 20 marker genes from each DE test were compiled together"
- [discussion] Marker rank aggregation (top 20 markers per DE test via robust rank aggregation; or significant markers via Lancaster combination): [evidence span withheld — anchor status: not_found]
- [readme] MASI utilizes robust marker identification to identify marker genes from reference data: "MASI utilizes robust marker idenfication to identify marker genes from reference data and transfers cell-type labels to target data through MACA."
- [readme] install Seurat and RobustRankAggreg separately in R: "##install Seurat and RobustRankAggreg separately in R"
