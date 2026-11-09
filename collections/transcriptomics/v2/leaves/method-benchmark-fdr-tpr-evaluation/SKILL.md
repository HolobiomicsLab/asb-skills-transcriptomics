---
name: method-benchmark-fdr-tpr-evaluation
description: Use when benchmarking DS analysis methods for multi-sample, multi-group
  scRNA-seq data against simulated ground truth, especially to compare FDR/TPR under
  local (per-subpopulation) vs. global BH p-value adjustment.
when_to_use_negative:
- The task is applying a single DS method to real scRNA-seq data rather than benchmarking
  multiple methods against simulated ground truth with known DS genes.
- There is no sample-level replication (one sample per condition) — sample-level FDR/TPR
  evaluation and pseudobulk testing across samples require multiple samples per group.
- The SingleCellExperiment lacks sample_id, group_id, and cluster_id metadata columns
  required by aggregateData/pbDS/mmDS.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: muscat
  role: Provides simData (anchored simulation), aggregateData (pseudobulk aggregation),
    pbDS (aggregation-based DS testing), and mmDS (cell-level mixed-model DS testing)
  repo: github.com/HelenaLC/muscat
- name: edgeR
  role: Pseudobulk differential engine run on aggregated counts via pbDS
- name: DESeq2
  role: Pseudobulk differential engine run on aggregated counts via pbDS
- name: limma-voom
  role: Pseudobulk differential engine run on aggregated counts via pbDS
- name: Snakemake
  role: Workflow manager built to run the simulation replicates for the benchmark
- name: R
  role: Implementation language for the muscat simulation and DS analysis pipeline
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-19894-4
    title: muscat detects subpopulation-specific state transitions from multi-sample
      multi-condition single-cell transcriptomics data
  source_content_hash: sha256:e74d965838a26be335e0b6484107e84872df51e5f7339c518033b40ff75448ff
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# method-benchmark-fdr-tpr-evaluation

## Summary

Benchmark 16 differential state (DS) analysis methods on muscat-simulated multi-sample multi-condition scRNA-seq data, computing false discovery rate (FDR) and true positive rate (TPR) per method against simulated ground truth, stratified by DS type (DE, DP, DM, DB) and by locally vs. globally Benjamini-Hochberg adjusted p-values.

## When to use

Use when benchmarking DS analysis methods for multi-sample, multi-group scRNA-seq data against simulated ground truth, especially to compare FDR/TPR under local (per-subpopulation) vs. global BH p-value adjustment.

## When NOT to use

- The task is applying a single DS method to real scRNA-seq data rather than benchmarking multiple methods against simulated ground truth with known DS genes.
- There is no sample-level replication (one sample per condition) — sample-level FDR/TPR evaluation and pseudobulk testing across samples require multiple samples per group.
- The SingleCellExperiment lacks sample_id, group_id, and cluster_id metadata columns required by aggregateData/pbDS/mmDS.

## Inputs

- example_sce reference SingleCellExperiment from muscat (with sample_id, group_id, cluster_id colData columns)
- simData simulation parameters: 3 samples per group, 3 subpopulations, 10% differential-state genes, 200 cells per subpopulation-sample
- Pseudobulk SummarizedExperiment from muscat::aggregateData (counts summed by cluster_id, sample_id)

## Outputs

- ds_method_benchmark_results.csv with FDR and TPR for all 16 methods stratified by DS category (DE, DP, DM, DB) and adjustment scope (local vs. global)
- Per-method adjusted p-values under local (p_adj.loc) and global BH correction at the 0.05 threshold

## How to apply

Anchor the simulation to muscat's example_sce reference and generate multi-sample multi-group data with simData using 3 samples per group, 3 subpopulations, 10% of genes set as differential state, and 200 cells per subpopulation-sample. Aggregate simulated counts to pseudobulk with muscat::aggregateData (by cluster_id and sample_id) and run muscat::pbDS across the aggregation-based engines (edgeR, DESeq2, limma-voom, limma-trend), then run muscat::mmDS for the cell-level mixed-model variants, covering all 16 surveyed method variants. Apply Benjamini-Hochberg correction (p.adjust, method='fdr') twice: locally within each cell type (p_adj.loc) and globally across all gene-subpopulation tests. Compute FDR and TPR at the adjusted p-value < 0.05 threshold for each method, stratified by DS category (DE, DP, DM, DB) and adjustment scope. The rationale is that the muscat authors built this anchored simulation specifically to evaluate the 16 DS methods, and found that global adjustment is very conservative — it systematically reduces both FDR and TPR relative to local adjustment — while aggregation-based methods showed stable high performance.

## Related tools

- **muscat** (Provides simData (anchored simulation), aggregateData (pseudobulk aggregation), pbDS (aggregation-based DS testing), and mmDS (cell-level mixed-model DS testing)) — github.com/HelenaLC/muscat
- **edgeR** (Pseudobulk differential engine run on aggregated counts via pbDS)
- **DESeq2** (Pseudobulk differential engine run on aggregated counts via pbDS)
- **limma-voom** (Pseudobulk differential engine run on aggregated counts via pbDS)
- **Snakemake** (Workflow manager built to run the simulation replicates for the benchmark)
- **R** (Implementation language for the muscat simulation and DS analysis pipeline)

## Examples

```
pb <- aggregateData(sce, assay = "counts", fun = "sum", by = c("cluster_id", "sample_id")); ds_pb <- pbDS(pb, method = "edgeR")
```

## Evaluation signals

- All 16 method × DS-category (DE, DP, DM, DB) × adjustment-scope (local vs. global) FDR/TPR entries are present in ds_method_benchmark_results.csv.
- Aggregation-based pseudobulk methods reproduce the paper's stable high performance with FDR controlled near the nominal 0.05 adjusted-p threshold.
- Global BH adjustment yields systematically lower TPR (and lower FDR) than local per-subpopulation adjustment for every method, matching the paper's reported conservative behavior.
- The script runs end-to-end from simData through p.adjust without errors on the specified simulation dimensions (3 samples/group, 3 subpopulations, 10% DS genes, 200 cells per subpopulation-sample).
- True positives align with the simulated ground-truth gene set of differential state genes.

## Limitations

- Global p-value adjustment is very conservative, reducing both FDR and TPR relative to local adjustment.
- Methods such as scDD and MAST were not specifically intended for the across-sample (multi-group multi-sample) situation.
- Mixed-model (MM) methods performed comparably but at higher computational cost, which may not be worth the flexibility.
- Cell-level methods show potentially strong bias in statistical power toward larger cell populations, with decreased power for rarer populations; the simulation framework does not yet accommodate an arbitrary number of groups.

## Evidence

- [intro] Simulation framework anchored to reference and 16 methods: "we developed a simulation framework, which is anchored to a reference dataset, that mimics various characteristics of scRNA-seq data and used it to evaluate 16 DS analysis methods"
- [methods] muscat scope of analysis: "`muscat` can be used to perform differrential detection (DD) analyses in multi-sample, multi-group, multi-(cell-)subpopulation scRNA-seq data"
- [discussion] Aggregation-based methods stable performance: "the tested aggregation-based DS methods were extremely fast and showed overall a stable high performance"
- [readme] README quick-guide commands: "# run pseudobulk (aggregation-based) DS analysis
ds_pb <- pbDS(pb, method = "edgeR")"
