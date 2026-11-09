---
name: scrnaseq-summary-statistics-construction
description: Use when scRNA-seq count data are available and gene-level DE effect
  sizes (beta coefficients and their variances) must be generated as input for iDEA's
  joint DE and gene set enrichment analysis.
when_to_use_negative:
- When only individual-level count data are available and a full joint model of raw
  scRNA-seq counts is required rather than summary-statistic integration.
- When the downstream analysis needs DE calls with individual-level modeling assumptions
  rather than effect-size borrowing across genes (iDEA makes no explicit modeling
  assumptions on individual-level data).
- When gene sets have coverage rate below min_precent_annot = 0.0025 or fewer than
  min_degene = 5 detected DE genes — they are filtered out by design.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: iDEA
  role: Consumes the constructed summary statistics via CreateiDEAObject; installs
    from GitHub
  repo: https://github.com/xzhoulab/iDEA
- name: MAST
  role: Standalone DE method producing beta coefficients and variances for each gene
- name: edgeR
  role: Standalone DE method applied with ZINB-WaVE observation weighting to produce
    gene-level summary statistics
  repo: https://github.com/statOmics/zinbwaveZinger
- name: zingeR
  role: Standalone DE method producing gene-level summary statistics for iDEA augmentation
  repo: https://github.com/statOmics/zinbwaveZinger
provenance:
  source_task_ids:
  - task_004
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-15298-6
    title: Integrative differential expression and gene set enrichment analysis using
      summary statistics for scRNA-seq studies
  source_content_hash: sha256:669c5e0c1252efb63515673369c7fea334e41a3da264edeaa7bb8588cf2372bd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xzhoulab/iDEA
  tools:
  - CreateiDEAObject
  - MAST
  - edgeR
  - iDEA
  - iDEA.BMA
  - iDEA.fit
  - zingeR
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# scrnaseq-summary-statistics-construction

## Summary

Produce gene-level differential expression summary statistics (beta coefficient and its variance) from scRNA-seq data using standalone DE tools (MAST, edgeR with ZINB-WaVE weighting, zingeR), formatted for downstream integration by iDEA. iDEA consumes only these summary statistics, avoiding explicit modeling of individual-level scRNA-seq data.

## When to use

Use when scRNA-seq count data are available and gene-level DE effect sizes (beta coefficients and their variances) must be generated as input for iDEA's joint DE and gene set enrichment analysis.

## When NOT to use

- When only individual-level count data are available and a full joint model of raw scRNA-seq counts is required rather than summary-statistic integration.
- When the downstream analysis needs DE calls with individual-level modeling assumptions rather than effect-size borrowing across genes (iDEA makes no explicit modeling assumptions on individual-level data).
- When gene sets have coverage rate below min_precent_annot = 0.0025 or fewer than min_degene = 5 detected DE genes — they are filtered out by design.

## Inputs

- scRNA-seq raw count matrix (e.g. simulated data with known DE genes at tau1 = 5, CR = 10%, or real 10X/Islam/Usoskin datasets)
- Per-gene DE summary statistics: beta coefficient and its variance, from MAST, edgeR + ZINB-WaVE, or zingeR
- Gene set annotation matrix (annotation_data; humanGeneSets.RData or mouseGeneSets.RData)

## Outputs

- summary_data: gene-level summary statistics table (beta and var(beta)) formatted for CreateiDEAObject
- iDEA object containing filtered genes (var(beta) < max_var_beta = 100) and gene sets passing min_precent_annot = 0.0025

## How to apply

Run each standalone DE method — MAST, edgeR with ZINB-WaVE observation weighting (via the zinbwaveZinger workflow), and zingeR — on the scRNA-seq count matrix to obtain a per-gene beta coefficient and its variance for every tested gene. Load the results with data(summary_data) and pair them with gene set annotations (data(annotation_data), humanGeneSets, or mouseGeneSets from the xzhoulab/iDEA repository). Build the analysis container with CreateiDEAObject(summary_data, annotation_data, max_var_beta = 100, min_precent_annot = 0.0025, num_core = 10), which filters genes by coefficient variance (cutoff max_var_beta = 100) and gene sets by coverage rate (CR = annotated genes / tested genes, default 0.0025). The rationale is that iDEA models all genes jointly by borrowing information across genes via DE effect-size distributional properties, so it needs only standardized summary statistics rather than raw counts, making the pipeline scalable and less susceptible to gene-gene correlations and dropout.

## Related tools

- **iDEA** (Consumes the constructed summary statistics via CreateiDEAObject; installs from GitHub) — https://github.com/xzhoulab/iDEA
- **MAST** (Standalone DE method producing beta coefficients and variances for each gene)
- **edgeR** (Standalone DE method applied with ZINB-WaVE observation weighting to produce gene-level summary statistics) — https://github.com/statOmics/zinbwaveZinger
- **zingeR** (Standalone DE method producing gene-level summary statistics for iDEA augmentation) — https://github.com/statOmics/zinbwaveZinger

## Examples

```
library(iDEA); data(summary_data); data(annotation_data); idea <- CreateiDEAObject(summary_data, annotation_data, max_var_beta = 100, min_precent_annot = 0.0025, num_core = 10)
```

## Evaluation signals

- All genes in summary_data carry a finite beta coefficient and variance; genes with var(beta) >= 100 are dropped by CreateiDEAObject.
- Each gene set in annotation_data has coverage rate (annotated genes / tested genes) >= 0.0025 after filtering.
- In simulations at tau1 = 5, CR = 10%, iDEA-augmented power at true FDR 5% should reach ~81% (zingeR), ~61% (MAST), ~83% (edgeR) vs. standalone 65%/52%/67%.
- Top-1500 DE gene list Jaccard index between methods should increase from 0.59 to 0.77 after iDEA augmentation.
- Applying iDEA.fit.null (gene-label permutation) yields a calibrated null distribution so iDEA.FDR-estimated FDR values are controlled near the nominal level.

## Limitations

- z-score modeling effectively assumes the prior distribution of true effect sizes depends on the standard errors and, subsequently, sample size.
- DE analysis performed on inferred cell types may lead to inflated DE test statistics with artificially smaller standard errors.
- No software version numbers for the compared DE tools are specified in the discussion, complicating exact reproduction.

## Evidence

- [intro] iDEA makes use of summary statistics output from existing DE tools and does not make explicit modeling assumptions on the individual-level scRNA-seq data: "iDEA makes use of summary statistics output from existing DE tools and does not make explicit modeling assumptions on the individual-level scRNA-seq data"
- [methods] max_var_beta filter: "max_var_beta: the cutoff of the variance of the coefficient of genes. Genes with variance smaller than 'max_var_beta' are maintained. Default is 100"
- [methods] min_precent_annot filter: "min_percent_annot: the threshold of coverage rate (CR), i.e., the number of annotated genes (gene set size) divided by the  number of tested genes. Default value is 0.0025."
- [methods] iDEA object creation: "The iDEA object is created by the function CreateiDEAObject."
- [readme] zingeR-DESeq2 preparation: "prepared by [zingeR-DESeq2](https://github.com/statOmics/zinbwaveZinger) method."
