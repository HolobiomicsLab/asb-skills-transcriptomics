---
name: pseudobulk-count-aggregation-single-cell
description: Use when you have a SingleCellExperiment with sample_id, group_id and
  cluster_id metadata and need sample-level inference of subpopulation-specific state
  changes across experimental conditions.
when_to_use_negative:
- The goal is cell-level mixed-model inference (mmDS) rather than pseudobulk analysis,
  e.g. when sample counts are too low to aggregate
- The data are not multi-sample multi-group scRNA-seq with replicate samples per condition
  — pseudobulk DS requires sample-level replication (e.g., four replicates per condition)
- The input is already aggregated (bulk-like) counts rather than single-cell counts,
  in which case aggregateData and cluster/sample identifiers are undefined
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: muscat
  role: Provides prepSCE, aggregateData (pseudobulk sum per cluster_id x sample_id),
    pbDS, and mds/plotMDS for the aggregation-based DS workflow
  repo: github.com/HelenaLC/muscat
- name: edgeR
  role: Bulk RNA-seq differential engine applied to the aggregated pseudobulk counts
    inside pbDS
- name: R
  role: Implementation environment for the muscat aggregation and DS analysis workflow
provenance:
  source_task_ids:
  - task_006
  - task_003
  - task_008
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-19894-4
    title: muscat detects subpopulation-specific state transitions from multi-sample
      multi-condition single-cell transcriptomics data
  source_content_hash: sha256:0df6e93faaae0bb1cd50f8cdc5be9a46195233178b0a0a097f6b9dbc9d20cff0
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
  - scater
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# pseudobulk-count-aggregation-single-cell

## Summary

Aggregate cell-level scRNA-seq counts into pseudobulk profiles per cell subpopulation and sample (sum of counts) so that bulk RNA-seq differential engines such as edgeR can be used for multi-sample, multi-group differential state (DS) analysis in muscat.

## When to use

Use when you have a SingleCellExperiment with sample_id, group_id and cluster_id metadata and need sample-level inference of subpopulation-specific state changes across experimental conditions.

## When NOT to use

- The goal is cell-level mixed-model inference (mmDS) rather than pseudobulk analysis, e.g. when sample counts are too low to aggregate
- The data are not multi-sample multi-group scRNA-seq with replicate samples per condition — pseudobulk DS requires sample-level replication (e.g., four replicates per condition)
- The input is already aggregated (bulk-like) counts rather than single-cell counts, in which case aggregateData and cluster/sample identifiers are undefined

## Inputs

- SingleCellExperiment with counts assay and colData columns sample_id, group_id, cluster_id (e.g., ExperimentHub EH2259 Kang PBMC IFN-β data or the mouse cortex snRNA-seq dataset)
- Optionally muscat example_sce for simulation-anchored benchmarks

## Outputs

- Pseudobulk count matrix (genes x cluster-sample) from aggregateData
- Per-cluster DS results (gene, cluster_id, logFC, p-value, locally adjusted p-value p_adj.loc) from pbDS
- Pseudobulk MDS representation colored by cluster_id and group_id

## How to apply

Standardize cell metadata with muscat::prepSCE so colData carries sample_id, group_id, and cluster_id, then call aggregateData(sce, assay='counts', fun='sum', by=c('cluster_id','sample_id')) to compute pseudobulks as the sum of counts per cluster-sample. Feed the resulting pseudobulk object to muscat::pbDS (default edgeR engine) to test condition effects (e.g., LPS vs. vehicle, IFN-β stimulation) within each subpopulation. Call genes as differential state hits at FDR < 0.05 (locally adjusted p_adj.loc) and |logFC| > 1 per cluster. Rationale: aggregation-based DS methods were 'extremely fast and showed overall a stable high performance' versus cell-level mixed models, which performed similarly but at higher computational cost. The same aggregation machinery also supports differential detection by switching fun='num.detected' for use with pbDD.

## Related tools

- **muscat** (Provides prepSCE, aggregateData (pseudobulk sum per cluster_id x sample_id), pbDS, and mds/plotMDS for the aggregation-based DS workflow) — github.com/HelenaLC/muscat
- **edgeR** (Bulk RNA-seq differential engine applied to the aggregated pseudobulk counts inside pbDS)
- **R** (Implementation environment for the muscat aggregation and DS analysis workflow)

## Examples

```
pb <- aggregateData(sce, assay = "counts", fun = "sum", by = c("cluster_id", "sample_id")); ds_pb <- pbDS(pb, method = "edgeR")
```

## Evaluation signals

- The pseudobulk MDS plot separates cell clusters along MDS1 and control vs. stimulated groups along MDS2 (Kang PBMC IFN-β data)
- Per-cluster DS hit counts at FDR < 5% and |logFC| > 1 match vignette-reported numbers
- LPS mouse cortex analysis recovers ~915 DS genes (FDR < 0.05, |logFC| > 1) in at least one subpopulation, with 751 in a single subpopulation and endothelial cells most strongly affected
- Output table contains gene and cluster_id columns plus logFC, p-value, and adjusted p-value (p_adj.loc) per cluster
- Aggregated pseudobulk methods reproduce stable high performance with FDR controlled at the nominal 0.05 threshold in simulated multi-sample data

## Limitations

- LogFC estimates can be attenuated depending on the scale of the data analyzed
- Statistical power is potentially biased toward larger cell populations, with decreased power for rarer populations
- Global (across all gene-subpopulation tests) p-value adjustment is very conservative and reduces both FDR and TPR compared to local per-subpopulation adjustment
- If using fun='num.detected', the aggregated values are binarized-detection (nonzero) counts, intended for differential detection (pbDD) rather than standard DS

## Evidence

- [discussion] Aggregation-based DS methods were extremely fast and showed overall stable high performance: "the tested aggregation-based DS methods were extremely fast and showed overall a stable high performance"
- [intro] pseudobulk via bulk engines: "pseudobulk counts were used as input to bulk RNA-seq differential engines, such as edgeR22, DESeq223, or limma-voom24,25."
- [readme] aggregateData quick guide: "pb <- aggregateData(sce, assay = "counts", fun = "sum", by = c("cluster_id", "sample_id"))"
- [methods] aggregation workflow methods: "We first load an example dataset and perform a pseudo-bulk differential state analysis"
- [readme] differential detection variant: "fun = "num.detected", # !!!
    by = c("cluster_id", "sample_id"))"
