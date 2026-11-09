---
name: cell-type-count-aggregation
description: Use when you have a cell-by-feature scRNA-seq or scATAC-seq count matrix
  with per-cell type labels and need across-cell-type expression/accessibility profiles
  for Spearman/k-medoids clustering or Kendall's Tb correlation validation.
when_to_use_negative:
- The data is already bulk RNA-seq/ATAC-seq or a pre-aggregated cell-type-level table
  — no per-cell-type averaging is needed
- The goal is per-cell analysis (clustering single cells rather than comparing across-cell-type
  feature profiles)
- The analysis needs spatial or proteomics/metabolomics layers — MOSim's single-cell
  simulation covers only scRNA-seq and scATAC-seq
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: MOSim
  role: Generates the simulated scRNA-seq/scATAC-seq count matrix that is aggregated
    to the average count per cell type
  repo: github.com/ConesaLab/MOSim
- name: R
  role: Implementation environment for aggregation, log(x+1) transformation, and downstream
    Spearman/k-medoids and Kendall's Tb validation
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbaf110
    title: 'MOSim: bulk and single-cell multilayer regulatory network simulator'
  source_content_hash: sha256:5b84c340f9a52e708669b2f96a73fafb5ee554c51bd17b7bd9c38b4c5ea6e98f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/ConesaLab/MOSim
  tools:
  - MOSim
  - R
  - Seurat
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# cell-type-count-aggregation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aggregates a simulated (or experimental) single-cell count matrix to the average count per cell type and applies a log(x+1) transformation, producing cell-type-level feature profiles suitable for downstream correlation- and clustering-based validation of multi-omics regulatory simulations.

## When to use

Use when you have a cell-by-feature scRNA-seq or scATAC-seq count matrix with per-cell type labels and need across-cell-type expression/accessibility profiles for Spearman/k-medoids clustering or Kendall's Tb correlation validation.

## When NOT to use

- The data is already bulk RNA-seq/ATAC-seq or a pre-aggregated cell-type-level table — no per-cell-type averaging is needed
- The goal is per-cell analysis (clustering single cells rather than comparing across-cell-type feature profiles)
- The analysis needs spatial or proteomics/metabolomics layers — MOSim's single-cell simulation covers only scRNA-seq and scATAC-seq

## Inputs

- Simulated or measured single-cell count matrix (scRNA-seq gene counts, optionally scATAC-seq peak counts) from sc_mosim
- Per-cell cell type labels (e.g. six cell types from the pbmc.multiome SeuratData seed dataset)
- Optional per-cell experimental group labels for downstream group-wise Kendall's Tb correlation

## Outputs

- Cell-type x feature matrix of average counts per cell type
- Log(x+1)-transformed cell-type profile matrix ready for Spearman's distance / k-medoids clustering or Kendall's Tb correlation

## How to apply

Take the simulated scRNA-seq (and optionally scATAC-seq) count matrix produced by MOSim's sc_mosim — the scRNA-seq layer carries scATAC-seq and transcription factor regulators — and group cells by their cell type label (six cell types in the pbmc.multiome validation). For each feature, compute the average count across cells within each cell type, yielding a cell-type x feature profile matrix. Apply a log(x+1) transformation to the aggregated profiles, as specified in the MOSim methods, to stabilize variance and make Spearman/Kendall profile comparisons meaningful. The aggregated, log-transformed matrix then feeds Spearman's distance (1 - Spearman correlation) plus k-medoids clustering (k=10) for co-expression pattern checks, or Kendall's Tb correlation within each experimental group for gene-peak regulatory validation.

## Related tools

- **MOSim** (Generates the simulated scRNA-seq/scATAC-seq count matrix that is aggregated to the average count per cell type) — github.com/ConesaLab/MOSim
- **R** (Implementation environment for aggregation, log(x+1) transformation, and downstream Spearman/k-medoids and Kendall's Tb validation)

## Examples

```
library(dplyr); agg <- aggregate(t(counts), by = list(cell_type = cell_types), FUN = mean); agg_log <- log1p(agg[, -1])
```

## Evaluation signals

- The aggregated matrix has exactly one column (or row) per cell type — e.g. six cell types for the pbmc.multiome seed — matching the sc_mosim simulation design
- Aggregated values equal the mean count over cells within each cell type, then log(x+1); all values are >= 0 after transformation
- Downstream k-medoids clustering (k=10, Spearman distance) reproduces the reported misassignment rate of ~3.3% against true simulated cell-type/DE-group labels
- Downstream Kendall's Tb validation reproduces ~79.5% of activator/repressor gene-peak interactions above absolute 0.7 and no-effect random pairs centred at ~0.32
- PCA on the log-transformed profiles confirms clustering aligns with the simulation settings

## Limitations

- Aggregation to mean profiles discards single-cell heterogeneity; it is intended for across-cell-type profile validation, not per-cell analysis
- Mean aggregation on raw counts followed by log(x+1) is a profile-shape transformation; correlations (Spearman, Kendall's Tb) are on across-cell-type profiles and do not test per-cell variance
- Applicable only to scRNA-seq and scATAC-seq layers — MOSim's single-cell simulation is currently restricted to these two modalities

## Evidence

- [methods] Log transform applied after aggregation: [evidence span withheld — anchor status: not_found]
