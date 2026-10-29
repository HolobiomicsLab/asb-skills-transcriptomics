---
name: mixed-model-differential-state-testing
description: Use when analyzing scRNA-seq with multiple samples, groups, and cell
  subpopulations and cell-level (mixed-model) DS testing is preferred over pseudobulk
  aggregation, e.g. to compare mmDS against pbDS or when per-cell modeling is desired.
when_to_use_negative:
- The analysis targets differential detection (DD) rather than differential state
  — use pbDD or pbDS with method='DD' instead
- Hypotheses should be re-weighted using independent bulk RNA-seq evidence — use bbhw
  on top of mmDS/pbDS results instead
- Computational cost is limiting and aggregation-based pseudobulk DS analysis (pbDS)
  performs adequately — the paper notes MM methods performed similarly well but at
  higher computational cost
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: muscat
  role: R package providing mmDS for cell-level mixed-model DS analysis in multi-sample
    multi-group scRNA-seq data
  repo: github.com/HelenaLC/muscat
- name: MAST
  role: Cell-level scRNA-seq differential method applied for comparison, though not
    specifically intended for the across-sample situation
- name: scDD
  role: Cell-level scRNA-seq differential distribution method applied for comparison
    in the 16-method benchmark
- name: edgeR
  role: Pseudobulk DS engine used via pbDS as the aggregation-based comparison to
    mixed models
- name: DESeq2
  role: Pseudobulk DS engine used via pbDS as an aggregation-based comparison to mixed
    models
- name: limma-voom
  role: Pseudobulk DS engine used via pbDS as an aggregation-based comparison to mixed
    models
- name: Snakemake
  role: Workflow manager used to run simulation replicates for benchmarking methods
    including mmDS
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-19894-4
    title: muscat detects subpopulation-specific state transitions from multi-sample
      multi-condition single-cell transcriptomics data
  source_content_hash: sha256:940225fdbf2a2e95fd8efd7abdee099af5fda7659aae56b1e2b48bebf99a4cd8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DESeq2
  - MAST
  - R
  - Snakemake
  - edgeR
  - limma-voom
  - muscat
  - scDD
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# mixed-model-differential-state-testing

## Summary

Cell-level differential state (DS) testing for multi-sample multi-condition scRNA-seq using mixed models via muscat's mmDS, which fits per-cluster cell-level models across samples as an alternative to pseudobulk aggregation.

## When to use

Use when analyzing scRNA-seq with multiple samples, groups, and cell subpopulations and cell-level (mixed-model) DS testing is preferred over pseudobulk aggregation, e.g. to compare mmDS against pbDS or when per-cell modeling is desired.

## When NOT to use

- The analysis targets differential detection (DD) rather than differential state — use pbDD or pbDS with method='DD' instead
- Hypotheses should be re-weighted using independent bulk RNA-seq evidence — use bbhw on top of mmDS/pbDS results instead
- Computational cost is limiting and aggregation-based pseudobulk DS analysis (pbDS) performs adequately — the paper notes MM methods performed similarly well but at higher computational cost

## Inputs

- SingleCellExperiment object with counts assay and colData columns sample_id, group_id, cluster_id
- Cluster/sample metadata defining multi-sample multi-group design

## Outputs

- Per-cluster DS testing results (gene-level statistics, p-values, BH-adjusted p-values) from mmDS
- Comparison table of TPR/FDR vs. method (e.g., simulation_cellnumber_trends.csv)

## How to apply

Prepare a SingleCellExperiment with sample_id, group_id, and cluster_id metadata columns as required by muscat. Run mixed models directly on cell-level measurements with mmDS(sce, method=...), optionally with method='dream', fitting models per cluster across samples to identify subpopulation-specific state changes. Call DS genes at BH-adjusted p < 0.05, and compare against aggregation-based pbDS results (edgeR, DESeq2, limma-voom engines) on aggregated counts. In the paper's simulation benchmark, mmDS was one of 16 DS methods evaluated on muscat's multi-sample simulations anchored to a reference dataset, including scenarios varying cells per subpopulation-sample (20, 100, 200, 400).

## Related tools

- **muscat** (R package providing mmDS for cell-level mixed-model DS analysis in multi-sample multi-group scRNA-seq data) — github.com/HelenaLC/muscat
- **MAST** (Cell-level scRNA-seq differential method applied for comparison, though not specifically intended for the across-sample situation)
- **scDD** (Cell-level scRNA-seq differential distribution method applied for comparison in the 16-method benchmark)
- **edgeR** (Pseudobulk DS engine used via pbDS as the aggregation-based comparison to mixed models)
- **DESeq2** (Pseudobulk DS engine used via pbDS as an aggregation-based comparison to mixed models)
- **limma-voom** (Pseudobulk DS engine used via pbDS as an aggregation-based comparison to mixed models)
- **Snakemake** (Workflow manager used to run simulation replicates for benchmarking methods including mmDS)

## Examples

```
ds_mm <- mmDS(sce, method = "dream")
```

## Evaluation signals

- mmDS returns gene-level DS results per cluster with BH-adjusted p-values; DS genes are called at adjusted p < 0.05
- In the cell-number benchmark, TPR improves with more cells for all methods including mmDS, and FDR control varies drastically with cell number
- mmDS performance is comparable to aggregation-based pbDS methods in TPR/FDR benchmarks, but at higher computational cost
- Workflow is runnable via the Snakemake pipeline for simulation replicates as described in the article

## Limitations

- MM methods have higher computational cost than aggregation-based methods, which 'may not be worth the flexibility they provide'
- FDR control instability with number of cells is not yet understood ('it would be of interest to further explore the origins of this instability')
- Methods including cell-level approaches 'were not specifically designed for the multi-group multi-sample problem'
- LogFC estimates may be attenuated for some combinations depending on the scale of the data analyzed

## Evidence

- [readme] mmDS for cell-level mixed-model DS analysis in multi-sample multi-group scRNA-seq: "Mixed models can be run directly on cell-level measurements, e.g.:

r
ds_mm <- mmDS(sce, method = "dream")
"
- [abstract] muscat provides DS analysis tools for multi-sample multi-group scRNA-seq: "Here, we surveyed methods to perform cross-condition differential state analyses, including cell-level mixed models and methods based on aggregated pseudobulk data."
- [intro] Benchmark scope includes cell-level methods alongside pseudobulk: "we applied well-known scRNA-seq methods, such as scDD33 and MAST34, although these methods were not specifically intended for the across-sample situation."
- [discussion] Discussion of computational cost tradeoff of mixed models: "While MM methods performed similarly well, their computational cost may not be worth the flexibility they provide"
- [methods] mmDS fits cell-level models per cluster as alternative to pbDS: "`bbhw` can be executed on top of any per-celltype differential state analysis as produced by `pbDS` or `mmDS`"
