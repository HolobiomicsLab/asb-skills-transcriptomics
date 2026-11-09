---
name: single-cell-pseudobulk-benchmarking
description: Use when evaluating whether different scRNA-seq DS methods (edgeR/DESeq2/limma-voom
  pseudobulk vs MAST/dream mixed models) agree on top-ranked detections at FDR < 0.05,
  or when benchmarking muscat's pbDS/mmDS engines on simulated multi-sample data.
when_to_use_negative:
- The data are already aggregated pseudobulk bulk-level counts without the cell-level
  sample_id/group_id/cluster_id metadata needed by muscat
- The goal is per-method power/FDR calibration against a known ground truth rather
  than between-method concordance of top-ranked detections
- Only a single sample per condition is available, so sample-level pseudobulk inference
  and replicate-wise set intersection are not meaningful
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: muscat
  role: R package providing pbDS (pseudobulk aggregation-based) and mmDS (mixed-model)
    DS analysis engines used to generate result tables across simulation replicates
  repo: github.com/HelenaLC/muscat
- name: pbDS
  role: Pseudobulk DS engine run on aggregateData output, e.g. with method='edgeR',
    to produce method-wise significant gene calls
  repo: github.com/HelenaLC/muscat
- name: mmDS
  role: Cell-level mixed-model DS engine (e.g. method='dream') providing the non-aggregation
    comparison family for concordance analysis
  repo: github.com/HelenaLC/muscat
- name: edgeR
  role: Bulk RNA-seq differential engine applied to pseudobulk counts inside pbDS;
    representative aggregation-based method being benchmarked
- name: Snakemake
  role: Workflow manager used to run the simulation replicates for all DS methods
    in the benchmark
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-19894-4
    title: muscat detects subpopulation-specific state transitions from multi-sample
      multi-condition single-cell transcriptomics data
  source_content_hash: sha256:898839f6792ea38ebaef32c33c5a1fe9a6d530e9535f1829221b1a855538b070
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Snakemake
  - edgeR
  - mmDS
  - muscat
  - pbDS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# single-cell-pseudobulk-benchmarking

## Summary

Benchmarking skill for differential state (DS) analysis of multi-sample multi-group scRNA-seq data: compare pseudobulk (pbDS) and cell-level mixed-model (mmDS) engines across simulation replicates by intersecting FDR<0.05 significant gene sets and assessing between-method concordance (e.g., UpSet plots as in the muscat Fig. 3 concordance analysis).

## When to use

Use when evaluating whether different scRNA-seq DS methods (edgeR/DESeq2/limma-voom pseudobulk vs MAST/dream mixed models) agree on top-ranked detections at FDR < 0.05, or when benchmarking muscat's pbDS/mmDS engines on simulated multi-sample data.

## When NOT to use

- The data are already aggregated pseudobulk bulk-level counts without the cell-level sample_id/group_id/cluster_id metadata needed by muscat
- The goal is per-method power/FDR calibration against a known ground truth rather than between-method concordance of top-ranked detections
- Only a single sample per condition is available, so sample-level pseudobulk inference and replicate-wise set intersection are not meaningful

## Inputs

- Simulated or real SingleCellExperiment objects with colData columns sample_id, group_id, cluster_id (as required by muscat)
- DS analysis result tables per method, replicate, cell subpopulation, and DS category (gene IDs with raw and BH-adjusted p-values and logFCs from pbDS and mmDS)

## Outputs

- Significant gene sets per method/replicate/subpopulation at p_adj.loc < 0.05
- Between-method intersection size matrix for UpSet plotting
- UpSet plot of method-wise intersections (Fig. 3-style concordance structure)
- Concordance summary classifying common hits as truly differential and isolated intersections as true/false discoveries

## How to apply

Run each DS engine over the five simulation replicates per DS category using muscat's pbDS (pseudobulk aggregation via aggregateData, e.g. edgeR) and mmDS (mixed models, e.g. dream) on the simulated SingleCellExperiment object. For each method, replicate, and cell subpopulation, extract genes with Benjamini-Hochberg-adjusted p-value < 0.05 (p_adj.loc computed via p.adjust with method='fdr'). Intersect the significant gene sets across replicates within each DS category and compute intersection sizes across all methods, grouping aggregation-based methods (edgeR, limma) separately from mixed-model methods. Build an UpSet plot ordered to highlight agreement among aggregation- and MM-based methods versus isolated, method-unique discoveries; validate by confirming high concordance among aggregation- and MM-based methods and that isolated intersections are largely false discoveries attributable to vstresiduals-based methods.

## Related tools

- **muscat** (R package providing pbDS (pseudobulk aggregation-based) and mmDS (mixed-model) DS analysis engines used to generate result tables across simulation replicates) — github.com/HelenaLC/muscat
- **pbDS** (Pseudobulk DS engine run on aggregateData output, e.g. with method='edgeR', to produce method-wise significant gene calls) — github.com/HelenaLC/muscat
- **mmDS** (Cell-level mixed-model DS engine (e.g. method='dream') providing the non-aggregation comparison family for concordance analysis) — github.com/HelenaLC/muscat
- **edgeR** (Bulk RNA-seq differential engine applied to pseudobulk counts inside pbDS; representative aggregation-based method being benchmarked)
- **Snakemake** (Workflow manager used to run the simulation replicates for all DS methods in the benchmark)

## Examples

```
pb <- aggregateData(sce, assay="counts", fun="sum", by=c("cluster_id","sample_id")); ds_pb <- pbDS(pb, method="edgeR"); ds_mm <- mmDS(sce, method="dream")
```

## Evaluation signals

- All extracted gene sets use Benjamini-Hochberg adjustment (p.adjust method='fdr') at p_adj.loc < 0.05, intersected across the five replicates per DS category
- UpSet plot reproduces the reported structure: high intersection sizes among aggregation-based (edgeR, limma) and MM-based methods
- The majority of common (shared) hits are truly differential genes under the simulation ground truth
- Most isolated, single-method-unique intersections are false discoveries, with a high proportion attributable to vstresiduals-based methods
- Result tables cover all 16 surveyed DS methods (or the intended subset) consistently across replicates and cell subpopulations

## Limitations

- While MM methods performed similarly well, their computational cost may not be worth the flexibility they provide
- LogFC estimates were attenuated for some method/data-scale combinations
- Methods such as scDD and MAST were not specifically intended for the across-sample situation, limiting fair comparison
- Instability of FDR control with the number of cells is not yet understood

## Evidence

- [readme] run pseudobulk (aggregation-based) DS analysis: "# run pseudobulk (aggregation-based) DS analysis
ds_pb <- pbDS(pb, method = "edgeR")"
- [readme] Mixed models run on cell-level data: "Mixed models can be run directly on cell-level measurements, e.g.:
ds_mm <- mmDS(sce, method = "dream")"
- [intro] 16 methods evaluated via simulation framework: "we developed a simulation framework, which is anchored to a reference dataset, that mimics various characteristics of scRNA-seq data and used it to evaluate 16 DS analysis methods"
- [discussion] Aggregation methods stable high performance: "the tested aggregation-based DS methods were extremely fast and showed overall a stable high performance"
- [discussion] MM methods similar performance higher cost: "While MM methods performed similarly well, their computational cost may not be worth the flexibility they provide"
