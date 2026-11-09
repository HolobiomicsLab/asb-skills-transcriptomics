---
name: true-positive-rate-false-discovery-rate-benchmarking
description: Use when evaluating how well DS detection methods (aggregation-based,
  mixed-model, cell-level) recover known differential state genes, e.g., how TPR and
  FDR control vary with number of cells per subpopulation-sample.
when_to_use_negative:
- When the goal is applied DS analysis on real data rather than method benchmarking
  against simulated ground truth
- When the simulation must accommodate an arbitrary number of groups or hierarchical-type
  genes — the muscat simulation framework does not yet support these
edam_operation: http://edamontology.org/operation_3226
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3170
tools:
- name: muscat
  role: Simulation framework, pseudobulk aggregation (aggregateData), pseudobulk DS
    testing (pbDS), and cell-level mixed models (mmDS)
  repo: github.com/HelenaLC/muscat
- name: edgeR
  role: Pseudobulk differential engine run on aggregated counts within pbDS
- name: DESeq2
  role: Pseudobulk differential engine run on aggregated counts within pbDS
- name: limma-voom
  role: Pseudobulk differential engine run on aggregated counts within pbDS
- name: MAST
  role: Cell-level method included in the 16-method comparison
- name: scDD
  role: Cell-level method included in the 16-method comparison
- name: Snakemake
  role: Workflow manager to run simulation replicates
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-19894-4
    title: muscat detects subpopulation-specific state transitions from multi-sample
      multi-condition single-cell transcriptomics data
  source_content_hash: sha256:698084026ceaf9647a255e44e98bc801dc904e13cfbeb56db6bafd7c55a151d4
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
  edam_operation: http://edamontology.org/operation_3226
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# true-positive-rate-false-discovery-rate-benchmarking

## Summary

Benchmark differential state (DS) analysis methods for multi-sample multi-group scRNA-seq data by computing true positive rate (TPR) and false discovery rate (FDR) against known simulated state genes, using muscat's simulation framework anchored to a reference dataset.

## When to use

Use when evaluating how well DS detection methods (aggregation-based, mixed-model, cell-level) recover known differential state genes, e.g., how TPR and FDR control vary with number of cells per subpopulation-sample.

## When NOT to use

- When the goal is applied DS analysis on real data rather than method benchmarking against simulated ground truth
- When the simulation must accommodate an arbitrary number of groups or hierarchical-type genes — the muscat simulation framework does not yet support these

## Inputs

- Reference scRNA-seq SingleCellExperiment with sample_id, group_id, cluster_id metadata columns (e.g., example_sce)
- Simulation scenario parameters (e.g., cells per subpopulation-sample: 20, 100, 200, 400)
- Simulation replicates with known true state genes

## Outputs

- Per-method, per-scenario TPR and FDR table (e.g., simulation_cellnumber_trends.csv)
- TPR/FDR-versus-cell-number plots summarizing performance across replicates

## How to apply

Generate DE-type simulations with the muscat multi-sample simulation framework, anchored to a reference scRNA-seq dataset, varying one scenario parameter (e.g., cells per subpopulation-sample at 20, 100, 200, 400) while holding others fixed. For each replicate, aggregate counts per sample-cluster with aggregateData and run pseudobulk DS testing via pbDS using edgeR, DESeq2 and limma-voom engines, plus cell-level mixed models via mmDS (scDD and MAST complete the 16-method comparison). Call differential state genes at BH-adjusted p < 0.05 for each method and scenario. Compute TPR and FDR per method and cell number against the known simulated state genes, summarizing across replicates, and assemble TPR/FDR-versus-parameter tables and plots. Run replicates through a Snakemake workflow to keep the benchmark reproducible.

## Related tools

- **muscat** (Simulation framework, pseudobulk aggregation (aggregateData), pseudobulk DS testing (pbDS), and cell-level mixed models (mmDS)) — github.com/HelenaLC/muscat
- **edgeR** (Pseudobulk differential engine run on aggregated counts within pbDS)
- **DESeq2** (Pseudobulk differential engine run on aggregated counts within pbDS)
- **limma-voom** (Pseudobulk differential engine run on aggregated counts within pbDS)
- **MAST** (Cell-level method included in the 16-method comparison)
- **scDD** (Cell-level method included in the 16-method comparison)
- **Snakemake** (Workflow manager to run simulation replicates)

## Examples

```
pb <- aggregateData(sce, assay = "counts", fun = "sum", by = c("cluster_id", "sample_id")); ds_pb <- pbDS(pb, method = "edgeR")
```

## Evaluation signals

- TPR improves with more cells for all tested methods
- Aggregation-based methods attain decent TPR by ~100 cells per subpopulation-sample
- FDR control varies drastically with number of cells across most methods
- Calls are made at BH-adjusted p < 0.05 consistently across methods and scenarios
- Results summarize stably across simulation replicates (output table populated for every method × scenario combination)

## Limitations

- Instability of FDR control with number of cells is not yet understood; the origins of this instability warrant further exploration
- The simulation framework does not yet accommodate an arbitrary number of groups or hierarchical-type genes
- Cell-level methods (scDD, MAST) were not specifically intended for the across-sample situation; MM methods perform similarly well but at higher computational cost

## Evidence

- [intro] Simulation anchored to reference dataset: "we developed a simulation framework, which is anchored to a reference dataset, that mimics various characteristics of scRNA-seq data and used it to evaluate 16 DS analysis methods"
- [intro] Snakemake workflow for replicates: "All methods tested are available within the muscat R package and a Snakemake35 workflow was built to run simulation replicates."
- [intro] Pseudobulk engines: "pseudobulk counts were used as input to bulk RNA-seq differential engines, such as edgeR22, DESeq223, or limma-voom24,25."
- [readme] aggregateData and pbDS commands: "pb <- aggregateData(sce, assay = "counts", fun = "sum", by = c("cluster_id", "sample_id"))"
- [discussion] Aggregation methods stable high performance: "the tested aggregation-based DS methods were extremely fast and showed overall a stable high performance"
- [discussion] FDR instability missing signal: "it would be of interest to further explore the origins of this instability"
- [discussion] Simulation framework limitations: "our simulation framework could be extended: (i) to accommodate an arbitrary number of groups"
