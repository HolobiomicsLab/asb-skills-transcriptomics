---
name: subpopulation-specific-response-interpretation
description: Use when analyzing multi-sample, multi-condition scRNA/snRNA-seq data
  with cell subpopulation annotations (sample_id, group_id, cluster_id) and the goal
  is to find genes and cell types showing condition-specific state changes at the
  sample level.
when_to_use_negative:
- When only two cells-populations (not samples) must be compared with no sample-level
  replication — cross-sample inference is the core premise of this pseudobulk DS approach
- When the input is already aggregated pseudobulk or bulk RNA-seq data rather than
  cell-level counts with cluster/sample/group metadata
- When the goal is differential detection (DD) rather than differential state — use
  aggregateData with fun="num.detected" and pbDD instead
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: muscat
  role: R package providing aggregateData for pseudobulk aggregation and pbDS/mmDS
    for subpopulation-specific differential state analysis
  repo: github.com/HelenaLC/muscat
- name: edgeR
  role: Bulk RNA-seq differential engine invoked via pbDS(method="edgeR") on pseudobulk
    counts to test LPS vs. vehicle within each subpopulation
- name: R
  role: Execution environment for the muscat pseudobulk DS workflow (Bioconductor
    installation via BiocManager)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-19894-4
    title: muscat detects subpopulation-specific state transitions from multi-sample
      multi-condition single-cell transcriptomics data
  source_content_hash: sha256:3cb3c898bbe396ae684596c5b31af3cab8d2645197d11d44d99770eddfaafd7d
schema_version: 0.2.0
metadata:
  license_tier: open
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
yes_rate: 0.625
n_claims: 8
---

# subpopulation-specific-response-interpretation

## Summary

Pseudobulk differential state (DS) analysis workflow from the muscat package that follows each cell subpopulation across samples and conditions (e.g., vehicle vs. LPS in mouse cortex snRNA-seq) to identify subpopulation-specific gene-state changes, using aggregateData for count sums and pbDS with edgeR as the differential engine.

## When to use

Use when analyzing multi-sample, multi-condition scRNA/snRNA-seq data with cell subpopulation annotations (sample_id, group_id, cluster_id) and the goal is to find genes and cell types showing condition-specific state changes at the sample level.

## When NOT to use

- When only two cells-populations (not samples) must be compared with no sample-level replication — cross-sample inference is the core premise of this pseudobulk DS approach
- When the input is already aggregated pseudobulk or bulk RNA-seq data rather than cell-level counts with cluster/sample/group metadata
- When the goal is differential detection (DD) rather than differential state — use aggregateData with fun="num.detected" and pbDD instead

## Inputs

- SingleCellExperiment object with colData columns sample_id, group_id, cluster_id and a counts assay (e.g., mouse cortex snRNA-seq, vehicle vs LPS, 4 replicates per condition)
- Subpopulation (cluster) assignments per cell
- Condition labels (e.g., LPS vs. vehicle)

## Outputs

- Per-subpopulation DS results table (gene, cluster_id, logFC, p-value, adjusted p-value), e.g., lps_ds_results.csv
- Set of differential state genes (FDR < 0.05, |logFC| > 1) per subpopulation
- Ranking of subpopulations by strength of treatment response

## How to apply

Load the SingleCellExperiment with colData columns sample_id, group_id, and cluster_id (e.g., the mouse frontal cortex snRNA-seq dataset with four vehicle and four LPS replicates). Compute pseudobulks as sum of counts per cluster_id-by-sample_id with muscat::aggregateData(assay="counts", fun="sum"). Run muscat::pbDS(pb, method="edgeR"), which feeds the pseudobulk counts to edgeR to test LPS vs. vehicle within each subpopulation. Filter results to differential state hits at FDR < 0.05 and |logFC| > 1; in the mouse cortex LPS case this recovers ~915 DS genes in at least one subpopulation, most detected in only a single subpopulation. Interpret the biology by checking for gene sets upregulated across all subpopulations and identifying the most strongly affected subpopulation (endothelial cells in the reference analysis). Finally, export per-subpopulation results (gene, cluster_id, logFC, p-value, adjusted p-value) to a table such as lps_ds_results.csv.

## Related tools

- **muscat** (R package providing aggregateData for pseudobulk aggregation and pbDS/mmDS for subpopulation-specific differential state analysis) — github.com/HelenaLC/muscat
- **edgeR** (Bulk RNA-seq differential engine invoked via pbDS(method="edgeR") on pseudobulk counts to test LPS vs. vehicle within each subpopulation)
- **R** (Execution environment for the muscat pseudobulk DS workflow (Bioconductor installation via BiocManager))

## Examples

```
pb <- aggregateData(sce, assay="counts", fun="sum", by=c("cluster_id","sample_id")); ds_pb <- pbDS(pb, method="edgeR")
```

## Evaluation signals

- The recovered DS set at FDR < 0.05 and |logFC| > 1 should be on the order of ~915 genes across at least one subpopulation, with ~751 detected in only a single subpopulation
- A distinct gene set should appear upregulated across all subpopulations (consensus cluster_id 3 shows upregulation)
- Endothelial cells should rank as the most strongly affected subpopulation in the LPS response
- Pseudobulk matrix dimensions should be genes x (cluster_id, sample_id) combinations: sum-of-counts aggregation per cluster-sample pair
- Output table must contain per-subpopulation gene, cluster_id, logFC, p-value, and adjusted p-value columns

## Limitations

- LogFC estimates may be attenuated for some combinations depending on the scale of the data analyzed
- Statistical power is potentially strongly biased toward larger cell populations, with decreased power for rarer populations
- Instability of FDR control with the number of cells is not yet understood; sequencing depth may be insufficient to access all cell-state transcriptional programs

## Evidence

- [readme] README pseudobulk aggregation: [evidence span withheld — anchor status: external_doc]
- [readme] README pbDS with edgeR: [evidence span withheld — anchor status: external_doc]
- [abstract] Abstract mouse cortex LPS motivation: "We analyzed scRNA-seq data from mouse cortex cells to uncover subpopulation-specific responses to lipopolysaccharide treatment"
- [intro] Intro pseudobulk DS application: "we applied pseudobulk DS analysis to unravel subpopulation-specific responses within brain cortex tissue from mice treated with lipopolysaccharide (LPS)."
