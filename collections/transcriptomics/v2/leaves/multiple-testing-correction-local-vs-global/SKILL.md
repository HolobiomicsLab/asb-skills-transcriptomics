---
name: multiple-testing-correction-local-vs-global
description: Use when benchmarking or choosing the scope of FDR control (per-cluster
  vs. global BH correction) for gene-subpopulation differential state tests produced
  by muscat pbDS/mmDS across multiple samples and conditions.
when_to_use_negative:
- Input is a bulk RNA-seq dataset with a single testing universe and no subpopulation
  (cluster_id) stratification, so there is no local-vs-global distinction to compare
- The task is differential detection (DD) or bulk-based hypothesis weighing (bbhw)
  with gBH bin-based correction rather than DS analysis across subpopulations
- Cells are compared between conditions without sample-level replication (no group_id
  across samples), which the muscat framework and its p-value scopes are not designed
  for
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: muscat
  role: Provides simData, aggregateData, pbDS, and mmDS whose raw p-values are adjusted
    locally (p_adj.loc) and globally to benchmark the two scopes
  repo: github.com/HelenaLC/muscat
- name: edgeR
  role: Aggregation-based pseudobulk DS engine feeding raw p-values into the local
    vs. global BH comparison
- name: DESeq2
  role: Aggregation-based pseudobulk DS engine feeding raw p-values into the local
    vs. global BH comparison
- name: limma-voom
  role: Aggregation-based pseudobulk DS engine feeding raw p-values into the local
    vs. global BH comparison
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-19894-4
    title: muscat detects subpopulation-specific state transitions from multi-sample
      multi-condition single-cell transcriptomics data
  source_content_hash: sha256:f6ad7cb2d4f7cc2b24111884ccffb018516953a39089e0ee3319250733142347
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
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# multiple-testing-correction-local-vs-global

## Summary

Skill for comparing locally (per-subpopulation) versus globally (across all gene-subpopulation tests) adjusted p-values in differential state (DS) analysis of multi-sample multi-subpopulation scRNA-seq data. It demonstrates that global Benjamini-Hochberg adjustment is systematically conservative, reducing both FDR and TPR relative to local adjustment.

## When to use

Use when benchmarking or choosing the scope of FDR control (per-cluster vs. global BH correction) for gene-subpopulation differential state tests produced by muscat pbDS/mmDS across multiple samples and conditions.

## When NOT to use

- Input is a bulk RNA-seq dataset with a single testing universe and no subpopulation (cluster_id) stratification, so there is no local-vs-global distinction to compare
- The task is differential detection (DD) or bulk-based hypothesis weighing (bbhw) with gBH bin-based correction rather than DS analysis across subpopulations
- Cells are compared between conditions without sample-level replication (no group_id across samples), which the muscat framework and its p-value scopes are not designed for

## Inputs

- SingleCellExperiment (sce) with sample_id, group_id, and cluster_id colData columns (e.g., muscat example_sce)
- Simulated multi-sample multi-group scRNA-seq counts from muscat::simData
- Pseudobulk count matrices from muscat::aggregateData (aggregated by cluster_id and sample_id)
- Raw per-gene p-value tables from pbDS (edgeR, DESeq2, limma-voom, limma-trend) and mmDS mixed-model variants

## Outputs

- Locally adjusted p-values (p_adj.loc) per cell type via p.adjust(method='fdr')
- Globally adjusted p-values across all gene-subpopulation tests
- FDR/TPR performance table per method x DS category (DE, DP, DM, DB) x adjustment scope, written to ds_method_benchmark_results.csv

## How to apply

Simulate multi-sample multi-group scRNA-seq data with muscat::simData anchored to the example_sce reference (3 samples per group, 3 subpopulations, 10% of genes set to differential state, 200 cells per subpopulation-sample), then aggregate counts to pseudobulk with muscat::aggregateData by cluster_id and sample_id. Run muscat::pbDS across the aggregation-based engines (edgeR, DESeq2, limma-voom, limma-trend) and muscat::mmDS for cell-level mixed models to obtain raw p-values for every gene-subpopulation test from all 16 method variants. Apply Benjamini-Hochberg correction twice on the same p-values: locally per cell type (p_adj.loc, p.adjust with method='fdr' within each subpopulation) and globally across all subpopulations at once. Threshold both adjusted vectors at adjusted p-value < 0.05, compute FDR and TPR stratified by DS category (DE, DP, DM, DB) and adjustment scope, and compare: global adjustment shifts results toward lower FDR at the cost of lower TPR, so the scope choice is a power/conservatism trade-off rather than a correctness issue.

## Related tools

- **muscat** (Provides simData, aggregateData, pbDS, and mmDS whose raw p-values are adjusted locally (p_adj.loc) and globally to benchmark the two scopes) — github.com/HelenaLC/muscat
- **edgeR** (Aggregation-based pseudobulk DS engine feeding raw p-values into the local vs. global BH comparison)
- **DESeq2** (Aggregation-based pseudobulk DS engine feeding raw p-values into the local vs. global BH comparison)
- **limma-voom** (Aggregation-based pseudobulk DS engine feeding raw p-values into the local vs. global BH comparison)

## Examples

```
p_adj.loc <- apply(pval_mat, 2, p.adjust, method = "fdr"); p_adj.glob <- p.adjust(pval_mat, method = "fdr"); ds_pb <- pbDS(aggregateData(sce, assay = "counts", fun = "sum", by = c("cluster_id", "sample_id")), method = "edgeR")
```

## Evaluation signals

- All 16 method x DS category x adjustment-scope (local vs. global) FDR/TPR entries are present in the output table ds_method_benchmark_results.csv
- Global adjustment yields systematically lower FDR AND lower TPR than local adjustment at the same adjusted p < 0.05 threshold (the conservative-shift invariant)
- Aggregation-based pseudobulk methods show stable high performance with FDR controlled at the nominal 0.05 level under local adjustment
- p.adjust is verified to be applied with method='fdr' once within each cluster (p_adj.loc) and once across the full gene-subpopulation test universe (global)
- The benchmark script runs end-to-end on simData output with 3 samples per group, 3 subpopulations, and 10% DS genes

## Limitations

- Global p-value adjustment is very conservative: it systematically reduces both FDR and TPR, so power loss — not misbehavior — is the expected effect of the global scope
- The muscat simulation framework does not yet accommodate an arbitrary number of groups, limiting how broadly the local/global comparison can be extended
- Instability of FDR control with varying numbers of cells is not yet understood and may confound FDR/TPR comparisons
- There is a potentially strong bias in statistical power toward larger cell populations, with decreased power for rarer populations

## Evidence

- [methods] we implemented the procedure from Hu, Zhao and Zhou [@Hu2010] in the `gBH` function: "we implemented the procedure from Hu, Zhao and Zhou [@Hu2010] in the `gBH` function"
- [readme] provides methods for *Differential State* (DS) analyses in scRNA-seq data: "provides methods for *Differential State* (DS) analyses in scRNA-seq data"
- [discussion] the tested aggregation-based DS methods were extremely fast and showed overall a stable high performance: "the tested aggregation-based DS methods were extremely fast and showed overall a stable high performance"
