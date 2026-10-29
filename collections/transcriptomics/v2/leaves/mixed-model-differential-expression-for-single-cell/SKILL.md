---
name: mixed-model-differential-expression-for-single-cell
description: Use when the goal is per-cell-type differential state testing directly
  on single-cell counts across multiple samples and conditions, especially when per-cell-level
  flexibility (e.g., mixed-model variance handling) is wanted instead of pseudobulk
  aggregation.
when_to_use_negative:
- When sample sizes are small per cluster-sample and pseudobulk aggregation with a
  bulk engine (edgeR, DESeq2, limma-voom) would give stable, fast, well-controlled
  FDR — the article notes MM methods' 'computational cost may not be worth the flexibility
  they provide'
- When the input is already aggregated pseudobulk count data; mmDS operates on cell-level
  measurements, not pseudobulk matrices
- When performing differential detection (DD) rather than differential state analysis
  — use pbDD/aggregateData with fun='num.detected' instead per the README
edam_operation: http://edamontology.org/operation_3577
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: muscat
  role: provides mmDS for cell-level mixed model differential state testing on multi-sample
    multi-group scRNA-seq
  repo: github.com/HelenaLC/muscat
- name: R
  role: implementation environment for muscat and the benchmark scripts
- name: Snakemake
  role: workflow engine used to run simulation replicates of the 16 DS methods including
    mmDS variants
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-19894-4
    title: muscat detects subpopulation-specific state transitions from multi-sample
      multi-condition single-cell transcriptomics data
  source_content_hash: sha256:52ac4bf62369b5fe62cf9ae4a65acb3f563f9dc9352401ac302c5c5b3d14894d
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
  edam_operation: http://edamontology.org/operation_3577
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# mixed-model-differential-expression-for-single-cell

## Summary

Fit cell-level mixed model differential state analysis on multi-sample, multi-group scRNA-seq data using muscat::mmDS, treating sample as a random effect while testing condition effects within each cell subpopulation. This provides a cell-level alternative to pseudobulk aggregation that performed comparably in benchmarks but at higher computational cost.

## When to use

Use when the goal is per-cell-type differential state testing directly on single-cell counts across multiple samples and conditions, especially when per-cell-level flexibility (e.g., mixed-model variance handling) is wanted instead of pseudobulk aggregation.

## When NOT to use

- When sample sizes are small per cluster-sample and pseudobulk aggregation with a bulk engine (edgeR, DESeq2, limma-voom) would give stable, fast, well-controlled FDR — the article notes MM methods' 'computational cost may not be worth the flexibility they provide'
- When the input is already aggregated pseudobulk count data; mmDS operates on cell-level measurements, not pseudobulk matrices
- When performing differential detection (DD) rather than differential state analysis — use pbDD/aggregateData with fun='num.detected' instead per the README

## Inputs

- SingleCellExperiment object with colData columns sample_id, group_id, cluster_id
- cell-level count assay (counts)
- method variant specification for mmDS (e.g., method = 'dream')

## Outputs

- per-gene DS test results (p-values, logFC) for each subpopulation
- locally (p_adj.loc) and globally BH-adjusted p-values
- FDR/TPR performance metrics when applied in simulation benchmarking

## How to apply

Prepare a SingleCellExperiment whose colData contains sample_id, group_id, and cluster_id columns. Run muscat::mmDS directly on cell-level measurements — no aggregation step is needed, unlike pbDS — selecting a mixed model engine such as 'dream' among the variants surveyed in the Crowell et al. benchmark. Test condition effects within each cluster (subpopulation) across samples, then apply Benjamini-Hochberg correction to the resulting p-values, either locally per subpopulation (p_adj.loc) or globally across all gene-subpopulation tests; local adjustment is preferred since global adjustment is very conservative and reduces TPR. Benchmark context: run mmDS on muscat::simData simulations (e.g., example_sce-anchored, 3 samples per group, 10% DS genes, 200 cells per subpopulation-sample) alongside the 16 pbDS method variants and compare FDR/TPR at adjusted p < 0.05 stratified by DS category (DE, DP, DM, DB).

## Related tools

- **muscat** (provides mmDS for cell-level mixed model differential state testing on multi-sample multi-group scRNA-seq) — github.com/HelenaLC/muscat
- **R** (implementation environment for muscat and the benchmark scripts)
- **Snakemake** (workflow engine used to run simulation replicates of the 16 DS methods including mmDS variants)

## Examples

```
ds_mm <- mmDS(sce, method = "dream")
```

## Evaluation signals

- All 16 method variants × DS category (DE, DP, DM, DB) × adjustment-scope (local vs global) FDR/TPR entries are present in the output table (ds_method_benchmark_results.csv)
- FDR is controlled near the nominal adjusted p < 0.05 threshold under local BH correction
- Global p-value adjustment systematically reduces both FDR and TPR relative to local adjustment, matching the paper's Fig. 2a finding
- mmDS results are computed on the same simulated data (via simData from example_sce) used for the pbDS pseudobulk variants, allowing direct comparison

## Limitations

- MM methods performed similarly well to aggregation-based methods but at higher computational cost
- LogFC estimates were attenuated for some combinations depending on the scale of the data analyzed
- Global BH adjustment across all gene-subpopulation tests is very conservative, reducing TPR
- Methods were not specifically designed for the multi-group multi-sample problem and focus on comparing sets of cells

## Evidence

- [readme] mmDS on cell-level data: "Mixed models can be run directly on cell-level measurements, e.g.:

ds_mm <- mmDS(sce, method = "dream")"
- [discussion] MM methods cost: "While MM methods performed similarly well, their computational cost may not be worth the flexibility they provide"
- [abstract] simulation framework: "To evaluate method performance, we developed a flexible simulation that mimics multi-sample scRNA-seq data."
- [results] global adjustment conservative: [evidence span withheld — anchor status: not_found]
- [intro] 16 methods evaluated: "we developed a simulation framework, which is anchored to a reference dataset, that mimics various characteristics of scRNA-seq data and used it to evaluate 16 DS analysis methods"
