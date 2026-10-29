---
name: time-dependent-covariate-modeling
description: Use when analyzing time-series scRNA-seq data where cell asynchrony must
  be corrected via pseudotime and the goal is to identify genes or cell subclusters
  whose abundance fluctuates significantly over pseudotime.
when_to_use_negative:
- Data are not time-series or no pseudotime/trajectory information is available —
  the time-dependent covariate requires ordered cells along a trajectory
- The analysis goal is static differential expression between predefined conditions
  rather than fluctuations along pseudotime (e.g. plain Monocle or tradeSeq-style
  DE suffices)
- Trajectory inference has not been performed and 'full'-mode preprocessing (gene
  filtering, scran normalization, root-type selection) is outside the user's scope
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scTITANS
  role: 'Implements the time-dependent covariate test: curve fitting along pseudotime,
    F-statistic computation, bootstrap null distribution, and q-value calling of significant
    genes and subclusters'
  repo: https://github.com/ZJUFanLab/scTITANS
- name: Monocle3
  role: 'Upstream trajectory inference: constructs the trajectory, orders cells by
    pseudotime via UMAP and Louvain clustering, supplying the covariate axis for the
    time-dependent test'
- name: scran
  role: Pool-and-deconvolute normalization of the filtered UMI count matrix prior
    to trajectory inference and time-dependent covariate analysis
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2021.07.016
    title: Identify differential genes and cell subclusters from time-series scRNA-seq
      data using scTITANS
  source_content_hash: sha256:61a3e28d1d340615d07e006673746b47831da28dc4ae7575f9bd546c4689c459
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
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# time-dependent-covariate-modeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

scTITANS' time-dependent covariate analysis fits a curve for each gene's relative abundance (and each subcluster's cell counts) along Monocle3-derived pseudotime, tests the complete model against a flat-line null with an F statistic, and assigns bootstrap-derived q-values to call time-varying genes and subclusters in time-series scRNA-seq data.

## When to use

Use when analyzing time-series scRNA-seq data where cell asynchrony must be corrected via pseudotime and the goal is to identify genes or cell subclusters whose abundance fluctuates significantly over pseudotime.

## When NOT to use

- Data are not time-series or no pseudotime/trajectory information is available — the time-dependent covariate requires ordered cells along a trajectory
- The analysis goal is static differential expression between predefined conditions rather than fluctuations along pseudotime (e.g. plain Monocle or tradeSeq-style DE suffices)
- Trajectory inference has not been performed and 'full'-mode preprocessing (gene filtering, scran normalization, root-type selection) is outside the user's scope

## Inputs

- pseudotime-ordered gene expression matrix (genes in rows, cells in columns) from Monocle3 trajectory inference
- cell metadata file (row cells) containing cell clusters and/or cell types
- gene metadata file (row genes, header must contain gene_short_name)
- root cluster name (or "none" if unknown) and fdr level (e.g. 0.01)

## Outputs

- SigGenes.fdr0.01.txt — significant genes with q-values below the fdr level
- SigClusters.fdr0.01.txt — significant cell subclusters with q-values below the fdr level (error message if cluster identification fails)

## How to apply

Load the pseudotime-ordered expression matrix (genes in rows, cells in columns) and cell metadata produced by upstream Monocle3 trajectory inference (Louvain clustering, UMAP). For each gene, fit a complete model along pseudotime using a p-dimensional linear basis (basis dimension chosen by singular value decomposition) and a flat-line null model, then compute the F statistic from the sum of squared residuals of both models (equations 3-4). Build the null distribution by bootstrapping: resample residuals from the alternative fit, add them back to the null fit, and form the per-gene p-value from the frequency that bootstrap statistics exceed the observed statistic (equation 5); convert p-values to q-values via the Storey-style pi0 estimator (equation 6) at fdr_level 0.01. Separately bin pseudotime, count cells of each subcluster per bin, fit complete versus flat null curves for cell numbers, and repeat the same F-statistic/bootstrap/q-value procedure to call significant subclusters. Write results to SigGenes.fdr0.01.txt and SigClusters.fdr0.01.txt; if cluster significance fails, an error message is emitted instead.

## Related tools

- **scTITANS** (Implements the time-dependent covariate test: curve fitting along pseudotime, F-statistic computation, bootstrap null distribution, and q-value calling of significant genes and subclusters) — https://github.com/ZJUFanLab/scTITANS
- **Monocle3** (Upstream trajectory inference: constructs the trajectory, orders cells by pseudotime via UMAP and Louvain clustering, supplying the covariate axis for the time-dependent test)
- **scran** (Pool-and-deconvolute normalization of the filtered UMI count matrix prior to trajectory inference and time-dependent covariate analysis)

## Examples

```
result = scTITANS_full(data_file,"none","none","none","F","","","none",0.01)
```

## Evaluation signals

- Both output files SigGenes.fdr0.01.txt and SigClusters.fdr0.01.txt exist and are returned without an error message from the cluster-identification step
- Every gene and subcluster listed in the output files carries a q-value below the 0.01 fdr level (smaller q-value = more significant difference between complete and null models)
- The observed F statistic per gene exceeds the bootstrap null distribution sufficiently often to yield a significant p-value under the resampled-residual scheme (equation 5)
- On the HSMM benchmark (GSE52529, 271 cells, two branches), the run completes with Monocle3-derived pseudotime as the covariate axis
- The basis dimension for the complete model is chosen by singular value decomposition, and the fitted curve differs significantly from the flat-line null

## Limitations

- If identification of significant clusters fails, scTITANS displays an error information instead of SigClusters.fdr0.01.txt
- Only Monocle and Monocle3 scored on the two-branch HSMM dataset; GPfates' trajectory was much less reasonable and Mpath provided trajectories only for cluster centroids, so the quality of the time-dependent analysis depends critically on the upstream TI method
- No software version numbers for scTITANS, Monocle3, or scran are stated in the article
- The root cell type may be unknown (pass "none"), which can affect pseudotime orientation and hence the fitted curves

## Evidence

- [methods] For each gene, fit a complete model along pseudotime using a p-dimensional linear basis for the fitted mean expression (basis dimension chosen by singular value decomposition) and fit a flat-line null model, then compute the F statistic from the sum of squared residuals of both models (equations 3-4).: [evidence span withheld — anchor status: not_found]
