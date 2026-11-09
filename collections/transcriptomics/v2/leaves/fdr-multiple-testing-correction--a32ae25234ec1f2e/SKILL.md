---
name: fdr-multiple-testing-correction
description: Use when calling significant differential state (DS) or differential
  detection genes across many cell clusters/genes in multi-sample multi-condition
  scRNA-seq data, where per-gene p-values must be adjusted to control FDR before hit
  calling.
when_to_use_negative:
- When no multi-hypothesis setting exists — correction is unnecessary for a single
  pre-specified hypothesis test
- 'For bbhw specifically: when no independent bulk RNA-seq dataset is available to
  create the bulk-data evidence bins that bbhw requires'
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: muscat
  role: Provides pbDS for pseudobulk DS testing with FDR adjustment, and bbhw/gBH
    for bulk-informed multiple-testing correction
  repo: github.com/HelenaLC/muscat
- name: edgeR
  role: Default bulk RNA-seq differential engine inside pbDS that produces the gene-level
    p-values submitted to FDR correction
- name: limma-voom
  role: Alternative bulk differential engine whose p-values can feed pseudobulk DS
    testing and correction
- name: R
  role: Language in which the muscat correction workflow is executed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-19894-4
    title: muscat detects subpopulation-specific state transitions from multi-sample
      multi-condition single-cell transcriptomics data
  source_content_hash: sha256:b60a8dc4163b99250ae43dd2491ff6f97823a5190d0a3c3bd1ab4159da62a1af
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - edgeR
  - limma-voom
  - muscat
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# fdr-multiple-testing-correction

## Summary

Multiple-testing false discovery rate (FDR) correction as used in the muscat differential state framework: adjusting per-cluster DS p-values (e.g., to FDR 5% with |logFC| > 1) and, optionally, leveraging independent bulk RNA-seq data via bulk-based hypothesis weighing (bbhw) bins and correction methods such as the gBH procedure.

## When to use

Use when calling significant differential state (DS) or differential detection genes across many cell clusters/genes in multi-sample multi-condition scRNA-seq data, where per-gene p-values must be adjusted to control FDR before hit calling.

## When NOT to use

- When no multi-hypothesis setting exists — correction is unnecessary for a single pre-specified hypothesis test
- For bbhw specifically: when no independent bulk RNA-seq dataset is available to create the bulk-data evidence bins that bbhw requires

## Inputs

- Gene-level raw p-values and logFCs from pbDS per cell cluster (cluster x gene result tables)
- Pseudobulk SingleCellExperiment produced by muscat::aggregateData (assay 'counts', fun 'sum')
- Optional: independent bulk RNA-seq p-values/logFCs used to bin hypotheses for bbhw
- Thresholds: adjusted p-value < 0.05 (FDR 5%) and |logFC| > 1

## Outputs

- Per-cluster table of significant DS genes at FDR 5% and |logFC| > 1
- Adjusted p-values (FDR-corrected) alongside raw p-values per gene and cluster
- Summary count table (e.g., pbds_ds_hits.csv) of DS hits per cluster

## How to apply

After running pseudobulk DS analysis with muscat::pbDS on aggregated pseudobulks (aggregateData with assay 'counts', fun 'sum', by cluster_id and sample_id), apply FDR correction to the per-cluster gene-level p-values and call hits at adjusted p-value < 0.05 (FDR 5%) and |logFC| > 1. When independent bulk RNA-seq data are available, muscat's bbhw (bulk-based hypothesis weighing) can increase power: hypotheses are grouped into different bins based on the bulk dataset and FDR methods are then applied within/after binning, with correction.method options such as 'sig' (bulk significance used as is, taking logFC direction into account) and 'combined'. For grouped hypotheses, muscat implements the Hu, Zhao and Zhou (2010) procedure in the gBH function. In the Kang PBMC IFN-β reproduction, corrected per-cluster hit counts are summarized to pbds_ds_hits.csv and validated against vignette-reported numbers.

## Related tools

- **muscat** (Provides pbDS for pseudobulk DS testing with FDR adjustment, and bbhw/gBH for bulk-informed multiple-testing correction) — github.com/HelenaLC/muscat
- **edgeR** (Default bulk RNA-seq differential engine inside pbDS that produces the gene-level p-values submitted to FDR correction)
- **limma-voom** (Alternative bulk differential engine whose p-values can feed pseudobulk DS testing and correction)
- **R** (Language in which the muscat correction workflow is executed)

## Examples

```
ds_pb <- pbDS(pb, method = "edgeR")
```

## Evaluation signals

- Per-cluster DS hit counts at adjusted p-value < 0.05 and |logFC| > 1 reproduce the muscat vignette-reported numbers on the Kang PBMC IFN-β dataset
- All adjusted p-values fall in [0,1] and are >= the corresponding raw p-values within each cluster's test family
- MDS plot of pseudobulk samples separates by cluster_id (MDS1) and group_id (MDS2), confirming the tested structure before correction
- Summary written to pbds_ds_hits.csv contains one hit count per cluster with no missing cluster rows

## Limitations

- FDR control can be unstable with the number of cells per group; the origins of this instability are not yet understood
- Statistical power is potentially strongly biased toward larger cell populations, with decreased power for rarer populations
- Cell-level and aggregation-based methods were not specifically designed for the multi-group multi-sample problem

## Evidence

- [methods] The general strategy is to group the hypotheses into different bins based on the bulk dataset, and then employ some False Discovery Rate (FDR) methods: [evidence span withheld — anchor status: not_found]
- [methods] gBH: "we implemented the procedure from Hu, Zhao and Zhou [@Hu2010] in the `gBH` function"
- [methods] bulk significance correction.method: "the bulk significance values are used as is, eventually taking the direction of the logFC into account"
- [readme] pbDS on pseudobulks: "# run pseudobulk (aggregation-based) DS analysis
ds_pb <- pbDS(pb, method = "edgeR")"
- [intro] pseudobulk counts as input to bulk engines: "pseudobulk counts were used as input to bulk RNA-seq differential engines, such as edgeR22, DESeq223, or limma-voom24,25."
