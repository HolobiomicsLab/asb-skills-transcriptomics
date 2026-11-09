---
name: cell-number-scaling-performance-evaluation
description: Use when designing or interpreting simulations that need the effect of
  cells-per-subpopulation-sample on differential state detection performance (TPR/FDR)
  quantified across aggregation-based and cell-level methods.
when_to_use_negative:
- When the question is about sensitivity to number of samples or groups rather than
  cells per subpopulation-sample (the simulation framework does not yet accommodate
  an arbitrary number of groups)
- When a real (non-simulated) dataset is being analyzed directly — TPR/FDR cannot
  be computed without known simulated state genes
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: muscat
  role: R package providing the multi-sample simulation framework, aggregateData,
    pbDS and mmDS used to run the cell-number scaling benchmark
  repo: https://github.com/HelenaLC/muscat
- name: edgeR
  role: Pseudobulk differential engine invoked through pbDS on aggregated counts
- name: DESeq2
  role: Pseudobulk differential engine invoked through pbDS on aggregated counts
- name: limma-voom
  role: Pseudobulk differential engine invoked through pbDS on aggregated counts
- name: MAST
  role: Cell-level method completing the 16-method comparison at each cell-number
    scenario
- name: scDD
  role: Cell-level method completing the 16-method comparison at each cell-number
    scenario
- name: Snakemake
  role: Workflow manager built to run the simulation replicates
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-19894-4
    title: muscat detects subpopulation-specific state transitions from multi-sample
      multi-condition single-cell transcriptomics data
  source_content_hash: sha256:3ff13617169e4960fbc71829e75788ac6e85d6b9ab1a86685f1a1b602a9a693e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HelenaLC/muscat
  tools:
  - DESeq2
  - MAST
  - R
  - Snakemake
  - edgeR
  - limma-voom
  - muscat
  - scDD
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# cell-number-scaling-performance-evaluation

## Summary

A simulation-based benchmarking skill that varies the number of cells per subpopulation-sample (e.g., 20, 100, 200, 400) in muscat's multi-sample DE-type simulations to quantify how TPR and FDR control of DS analysis methods (pseudobulk edgeR/DESeq2/limma-voom, mmDS, scDD, MAST) scale with cell counts.

## When to use

Use when designing or interpreting simulations that need the effect of cells-per-subpopulation-sample on differential state detection performance (TPR/FDR) quantified across aggregation-based and cell-level methods.

## When NOT to use

- When the question is about sensitivity to number of samples or groups rather than cells per subpopulation-sample (the simulation framework does not yet accommodate an arbitrary number of groups)
- When a real (non-simulated) dataset is being analyzed directly — TPR/FDR cannot be computed without known simulated state genes

## Inputs

- Reference scRNA-seq SingleCellExperiment with sample_id, group_id and cluster_id colData columns (e.g., mouse cortex LPS dataset)
- muscat simulation scenario parameters (DE-type, cells per subpopulation-sample in {20, 100, 200, 400}, other parameters fixed)
- Simulation replicate specifications for the Snakemake workflow

## Outputs

- simulation_cellnumber_trends.csv (TPR/FDR-versus-cell-number table per method)
- TPR and FDR summary plots versus number of cells per subpopulation-sample
- Per-method DS gene calls at BH-adjusted p < 0.05 for each cell-number scenario

## How to apply

In R with muscat, generate DE-type simulations anchored to a reference scRNA-seq dataset (e.g., the mouse cortex LPS data) using muscat's multi-sample simulation framework, varying cells per subpopulation-sample over 20, 100, 200 and 400 while holding all other parameters fixed. For each replicate, aggregate counts per sample-cluster with aggregateData and run pseudobulk DS testing via pbDS using the edgeR, DESeq2 and limma-voom engines, and fit cell-level models with mmDS (scDD and MAST complete the 16-method comparison). Call differential state genes at BH-adjusted p < 0.05 for each method and cell-number scenario, then compute TPR and FDR per method and cell number against the known simulated state genes, summarizing across replicates. Rationale: the paper showed TPRs improve with more cells for all methods, aggregation-based methods reach decent performance with ~100 cells per subpopulation-sample, while FDR control varies drastically with cell number — an instability the authors note is not yet understood. A Snakemake workflow was built to run the simulation replicates.

## Related tools

- **muscat** (R package providing the multi-sample simulation framework, aggregateData, pbDS and mmDS used to run the cell-number scaling benchmark) — https://github.com/HelenaLC/muscat
- **edgeR** (Pseudobulk differential engine invoked through pbDS on aggregated counts)
- **DESeq2** (Pseudobulk differential engine invoked through pbDS on aggregated counts)
- **limma-voom** (Pseudobulk differential engine invoked through pbDS on aggregated counts)
- **MAST** (Cell-level method completing the 16-method comparison at each cell-number scenario)
- **scDD** (Cell-level method completing the 16-method comparison at each cell-number scenario)
- **Snakemake** (Workflow manager built to run the simulation replicates)

## Examples

```
pb <- aggregateData(sce, assay = "counts", fun = "sum", by = c("cluster_id", "sample_id")); ds_pb <- pbDS(pb, method = "edgeR")
```

## Evaluation signals

- TPR improves monotonically (or near-monotonically) with more cells per subpopulation-sample across all methods
- Aggregation-based (pseudobulk) methods attain decent TPR by ~100 cells per subpopulation-sample
- FDR control varies drastically with cell number — a stable, low FDR across all cell counts should NOT be expected
- DS calls correspond to the known simulated state genes at BH-adjusted p < 0.05, enabling exact TPR/FDR recovery
- Output table simulation_cellnumber_trends.csv contains one TPR/FDR row per method per cell-number scenario, summarized across replicates

## Limitations

- FDR instability with number of cells is not yet understood; the authors state 'it would be of interest to further explore the origins of this instability'
- The muscat simulation framework does not yet accommodate an arbitrary number of groups or hierarchical-type genes
- Cell-level mixed models perform similarly well but at much higher computational cost than aggregation-based methods

## Evidence

- [intro] Simulation framework anchored to reference dataset: "we developed a simulation framework, which is anchored to a reference dataset, that mimics various characteristics of scRNA-seq data and used it to evaluate 16 DS analysis methods"
- [intro] Snakemake workflow for replicates: "All methods tested are available within the muscat R package and a Snakemake35 workflow was built to run simulation replicates."
- [discussion] FDR instability caveat: "it would be of interest to further explore the origins of this instability"
- [readme] aggregateData pseudobulk command: "pb <- aggregateData(sce, assay = "counts", fun = "sum", by = c("cluster_id", "sample_id"))"
- [readme] mmDS mixed-model entry point: "ds_mm <- mmDS(sce, method = "dream")"
