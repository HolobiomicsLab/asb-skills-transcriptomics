---
name: bootstrap-null-distribution-qvalue-calculation
description: Use when calling differentially expressed genes or subclusters along
  pseudotime in scTITANS and you need calibrated p-values/q-values for time-dependent
  expression via a bootstrap null instead of a parametric assumption.
when_to_use_negative:
- The analysis goal is a simple two-condition differential test (e.g., DESeq2-style)
  with no pseudotime/trajectory structure — the time-dependent covariate model is
  designed for time-series scRNA-seq ordered along pseudotime
- Trajectory inference has not been performed and no pseudotime is available, so gene
  curves along pseudotime cannot be fitted
- The user only wants trajectory construction itself (e.g., choosing among Monocle,
  Monocle3, Slingshot, TSCAN) rather than downstream differential testing along the
  inferred trajectory
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scTITANS
  role: Implements the curve fitting, F-statistic computation, bootstrap null construction,
    and q-value conversion inside scTITANS_full / scTITANS_partial; outputs SigGenes
    and SigClusters files at the chosen fdr_level
  repo: https://github.com/ZJUFanLab/scTITANS
- name: Monocle3
  role: 'Upstream: constructs the trajectory (Louvain clustering, UMAP) and computes
    per-cell pseudotime on which gene curves are fitted and the bootstrap null is
    built'
- name: scran
  role: 'Upstream: pool-and-deconvolute normalization of the filtered count matrix
    so fitted expression curves and their residuals are computed on normalized values'
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2021.07.016
    title: Identify differential genes and cell subclusters from time-series scRNA-seq
      data using scTITANS
  source_content_hash: sha256:27b27aca1c8f915e192f71e0366c507627e2ca7830e626f28d96f23a3ca95255
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ZJUFanLab/scTITANS
  repo_url: https://github.com/ZJUFanLab/scTITANS
  tools:
  - Monocle3
  - scTITANS
  - scran
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# bootstrap-null-distribution-qvalue-calculation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A statistical step within the scTITANS workflow that derives significance of time-dependent genes (and cell subclusters) along pseudotime by fitting a curve per gene, computing an F statistic on residuals, building a bootstrap null distribution from residuals re-sampled from the alternative fit, and converting p-values to q-values at a user-specified FDR level.

## When to use

Use when calling differentially expressed genes or subclusters along pseudotime in scTITANS and you need calibrated p-values/q-values for time-dependent expression via a bootstrap null instead of a parametric assumption.

## When NOT to use

- The analysis goal is a simple two-condition differential test (e.g., DESeq2-style) with no pseudotime/trajectory structure — the time-dependent covariate model is designed for time-series scRNA-seq ordered along pseudotime
- Trajectory inference has not been performed and no pseudotime is available, so gene curves along pseudotime cannot be fitted
- The user only wants trajectory construction itself (e.g., choosing among Monocle, Monocle3, Slingshot, TSCAN) rather than downstream differential testing along the inferred trajectory

## Inputs

- Gene expression matrix (rows = genes, columns = cells), with per-cell pseudotime from Monocle3 trajectory inference
- scran pool-and-deconvolute normalized counts (scTITANS_full called with indication = 'normalized', or 'filtered'/'none' as appropriate)
- fdr_level threshold (e.g., 0.01) supplied as the final argument to scTITANS_full or scTITANS_partial
- Optional gene_metadata file with a gene_short_name column and cell_metadata with cluster/cell-type annotations (scTITANS_partial)

## Outputs

- Per-gene and per-subcluster p-values derived from the bootstrap null distribution
- q-values (FDR-adjusted) at the chosen fdr_level
- SigGenes.fdr<level>.txt — significant (time-dependent) genes
- SigClusters.fdr<level>.txt — significant cell subclusters whose counts vary along pseudotime

## How to apply

After Monocle3 (Louvain clustering, UMAP) has computed pseudotime for each cell, fit a curve for each gene's dynamic expression along pseudotime under the complete model and compare against a flat null model by computing the F statistic on residuals. Build the null distribution of this statistic by bootstrapping residuals re-sampled from the alternative fit, then derive p-values from the empirical null and convert them to q-values. Apply the same time-dependent covariate analysis to cell subcluster counts across pseudotime bins. Select the q-value threshold via the fdr_level argument (the article and README examples use 0.01); results are written as SigGenes.fdr0.01.txt and SigClusters.fdr0.01.txt. The rationale is that the bootstrap null captures the sampling variability of the fitted curve without relying on a closed-form null distribution.

## Related tools

- **scTITANS** (Implements the curve fitting, F-statistic computation, bootstrap null construction, and q-value conversion inside scTITANS_full / scTITANS_partial; outputs SigGenes and SigClusters files at the chosen fdr_level) — https://github.com/ZJUFanLab/scTITANS
- **Monocle3** (Upstream: constructs the trajectory (Louvain clustering, UMAP) and computes per-cell pseudotime on which gene curves are fitted and the bootstrap null is built)
- **scran** (Upstream: pool-and-deconvolute normalization of the filtered count matrix so fitted expression curves and their residuals are computed on normalized values)

## Examples

```
result = scTITANS_full(data_file,"none","none","none","F","","","none",0.01)
```

## Evaluation signals

- Running scTITANS_full(...,0.01) returns exactly the two named result files SigGenes.fdr0.01.txt and SigClusters.fdr0.01.txt without an error message from the cluster-identification step
- All reported q-values are ≤ the specified fdr_level (0.01) in the SigGenes/SigClusters output files
- Each significant gene has a fitted curve whose F statistic computed on residuals was compared against the bootstrap (resampled-residual) null, so p-values should follow the empirical null distribution rather than a parametric shape
- Significant genes should exhibit visible fluctuation along Monocle3 pseudotime; flat genes should not pass the q-value threshold
- The same time-dependent covariate machinery applied to subcluster counts should yield non-empty SigClusters output when subcluster counts vary across pseudotime bins

## Limitations

- No software version numbers for Monocle3, scran, or scTITANS are stated, so exact reproducibility of the bootstrap null is not guaranteed
- Significance of subclusters depends on the clustering step succeeding — the README notes that 'If something wrong happens in identifying significant clusters, an error information will be displayed'
- The procedure assumes reliable pseudotime from the trajectory inference step; on HSMM (GSE52529), only Monocle and Monocle3 produced scored, reasonable two-branch trajectories (GPfates was much less reasonable and Mpath only covered cluster centroids), so an inadequate TI method undermines the q-values
- scTITANS is specific to time-series scRNA-seq data where correcting cell asynchrony via pseudotime is the goal

## Evidence

- [methods] The null distribution of the statistic was calculated through a method named bootstrap: "The null distribution of the statistic was calculated through a method named bootstrap [41]."
- [methods] curve fitting of gene expression along pseudotime: "a curve is fitted for the dynamic expression of each gene along the pseudotime"
- [methods] time-dependent covariate on genes and cell numbers: "a time-dependent covariate [37] was introduced to evaluate the fluctuations of genes and cell numbers along pseudotime."
- [readme] expected output files at fdr 0.01: "The above two command will both return two txt files named "SigGenes.fdr0.01.txt" and "SigClusters.fdr0.01.txt", respectively."
- [results] only Monocle and Monocle3 scored on HSMM: "only Monocle, Monocle3, GPfates, and Mpath successfully reconstructed the two branches implicated in the 271 cells"
