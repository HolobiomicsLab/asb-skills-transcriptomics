---
name: multi-sample-multi-condition-scrna-seq-analysis
description: Use when scRNA-seq or snRNA-seq data span multiple biological samples
  and experimental groups (e.g., vehicle vs LPS-treated mice) and the goal is cell-subpopulation-level
  differential state inference rather than across-cell comparisons.
when_to_use_negative:
- Data are single-sample or single-group per condition such that sample-level inference
  (replicates) is impossible; pseudobulk DS requires multiple samples per group.
- The question is differential detection (DD) per cell rather than differential state
  across conditions; use muscat's pbDD / fun='num.detected' workflow instead.
- Cell-level mixed-model flexibility (mmDS with e.g. dream) is required despite higher
  cost; the aggregation-based pbDS route assumes pseudobulk sums are the appropriate
  unit of inference.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: muscat
  role: Core package providing aggregateData for pseudobulk summation by cluster_id/sample_id
    and pbDS for aggregation-based differential state testing
  repo: https://github.com/HelenaLC/muscat
- name: edgeR
  role: Bulk RNA-seq differential engine invoked by pbDS (method='edgeR') on the pseudobulk
    counts to fit group contrasts per subpopulation
- name: R
  role: Runtime environment in which the muscat/edgeR pseudobulk DS workflow is executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-19894-4
    title: muscat detects subpopulation-specific state transitions from multi-sample
      multi-condition single-cell transcriptomics data
  source_content_hash: sha256:b50e9f8280fd604d76e561f83670a65b3c84ffa9636433569384099c7c5331df
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HelenaLC/muscat
  tools:
  - R
  - edgeR
  - muscat
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# multi-sample-multi-condition-scrna-seq-analysis

## Summary

Pseudobulk differential state (DS) analysis of multi-sample, multi-condition, multi-subpopulation scRNA-seq/snRNA-seq data using muscat: single-cell counts are aggregated to pseudobulk sums per sample-by-subpopulation and tested with bulk RNA-seq engines such as edgeR to identify subpopulation-specific state transitions.

## When to use

Use when scRNA-seq or snRNA-seq data span multiple biological samples and experimental groups (e.g., vehicle vs LPS-treated mice) and the goal is cell-subpopulation-level differential state inference rather than across-cell comparisons.

## When NOT to use

- Data are single-sample or single-group per condition such that sample-level inference (replicates) is impossible; pseudobulk DS requires multiple samples per group.
- The question is differential detection (DD) per cell rather than differential state across conditions; use muscat's pbDD / fun='num.detected' workflow instead.
- Cell-level mixed-model flexibility (mmDS with e.g. dream) is required despite higher cost; the aggregation-based pbDS route assumes pseudobulk sums are the appropriate unit of inference.

## Inputs

- SingleCellExperiment object with raw counts and colData columns sample_id, group_id, cluster_id
- Multi-sample, multi-condition scRNA-seq/snRNA-seq dataset (e.g., mouse frontal cortex snRNA-seq, vehicle vs LPS, 4 replicates per condition)

## Outputs

- Per-subpopulation DS results table (gene, cluster_id, logFC, p-value, adjusted p-value), e.g., lps_ds_results.csv
- Filtered DS gene list at FDR < 0.05 and |logFC| > 1 (e.g., ~915 genes across at least one subpopulation)

## How to apply

Build a SingleCellExperiment with colData columns 'sample_id', 'group_id', and 'cluster_id'. Compute pseudobulks as the sum of counts per cluster_id-by-sample_id combination using muscat::aggregateData, then run muscat::pbDS with method='edgeR' to test each group contrast (e.g., LPS vs vehicle) within each subpopulation. In the mouse cortex LPS snRNA-seq example (four replicates per condition), hits were filtered to FDR < 0.05 and |logFC| > 1, yielding ~915 DS genes in at least one subpopulation, 751 unique to a single subpopulation. Aggregation-based (pseudobulk) methods were favored because benchmarks showed they are extremely fast with stable, high performance compared to cell-level mixed models (mmDS), which incur higher computational cost. Write per-subpopulation results (gene, cluster_id, logFC, p-value, adjusted p-value) to CSV for downstream review.

## Related tools

- **muscat** (Core package providing aggregateData for pseudobulk summation by cluster_id/sample_id and pbDS for aggregation-based differential state testing) — https://github.com/HelenaLC/muscat
- **edgeR** (Bulk RNA-seq differential engine invoked by pbDS (method='edgeR') on the pseudobulk counts to fit group contrasts per subpopulation)
- **R** (Runtime environment in which the muscat/edgeR pseudobulk DS workflow is executed)

## Examples

```
pb <- aggregateData(sce, assay="counts", fun="sum", by=c("cluster_id","sample_id")); ds_pb <- pbDS(pb, method="edgeR")
```

## Evaluation signals

- Recovered DS gene count is on the order of ~915 genes at FDR < 0.05 and |logFC| > 1 in at least one subpopulation for the cortex LPS dataset.
- Roughly 751 of the DS genes are detected in only a single subpopulation, consistent with subpopulation-specific state transitions.
- Consensus cluster (cluster_id) 3 shows upregulation across subpopulations and endothelial cells exhibit the strongest response.
- Output table schema contains gene, cluster_id, logFC, p-value, and adjusted p-value columns per subpopulation.
- pseudobulk object contains one summed-count column per cluster_id-by-sample_id combination (8 samples x subpopulations in the cortex dataset).

## Limitations

- Statistical power is potentially strongly biased toward larger cell populations, with decreased power for rarer populations.
- LogFC estimates can be attenuated depending on the scale of the data analyzed.
- FDR control shows instability with the number of cells, the origin of which is not yet understood.
- Cell-level mixed-model alternatives (mmDS) perform similarly but at substantially higher computational cost.

## Evidence

- [abstract] Mouse cortex LPS application of pseudobulk DS analysis: "We analyzed scRNA-seq data from mouse cortex cells to uncover subpopulation-specific responses to lipopolysaccharide treatment"
- [intro] Pseudobulk counts as input to bulk engines: "pseudobulk counts were used as input to bulk RNA-seq differential engines, such as edgeR22, DESeq223, or limma-voom24,25."
- [discussion] Benchmark performance of aggregation-based methods: "the tested aggregation-based DS methods were extremely fast and showed overall a stable high performance"
- [readme] README quick-guide pseudobulk commands: "pb <- aggregateData(sce, assay = "counts", fun = "sum", by = c("cluster_id", "sample_id"))"
- [methods] muscat scope statement: "`muscat` can be used to perform differrential detection (DD) analyses in multi-sample, multi-group, multi-(cell-)subpopulation scRNA-seq data"
