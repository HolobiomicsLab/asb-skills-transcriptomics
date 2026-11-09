---
name: bootstrap-null-distribution-estimation
description: Use when identifying genes or cell subclusters that vary significantly
  along pseudotime in time-series scRNA-seq data and a nonparametric null distribution
  is needed for a curve-vs-flat-line F statistic.
when_to_use_negative:
- Data lacks a trajectory/pseudotime ordering (scTITANS depends on pseudotime from
  Monocle3 trajectory inference to define the time-dependent covariate).
- The analysis goal is static two-condition differential expression without a time-series
  or pseudotime dimension — standard DE tools are more appropriate.
- Only raw, unprocessed data is available and only the bootstrap step is needed —
  scTITANS in 'full' mode would rerun preprocessing; use scTITANS_full instead of
  the partial/statistical stage.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2269
tools:
- name: scTITANS
  role: Implements the complete-versus-flat null model fitting, residual bootstrap,
    F-statistic evaluation, and Storey-style q-value estimation for genes and cell
    subclusters along pseudotime
  repo: https://github.com/ZJUFanLab/scTITANS
- name: Monocle3
  role: Upstream supplier of trajectory and per-cell pseudotime values that define
    the time-dependent covariate for the statistical test
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2021.07.016
    title: Identify differential genes and cell subclusters from time-series scRNA-seq
      data using scTITANS
  source_content_hash: sha256:441eb2ac02e02537d8dc6e6ca0fce76983ffa366c78e14a5a7a22a2938a964f3
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
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# bootstrap-null-distribution-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

scTITANS builds per-gene (and per-subcluster) null distributions by bootstrapping residuals from a fitted curve along Monocle3 pseudotime, turning an F statistic comparing a complete model to a flat-line null into an empirical p-value, then a Storey-style q-value at FDR 0.01. This is how pseudotime is converted into a statistical test for time-dependent genes and cell subclusters.

## When to use

Use when identifying genes or cell subclusters that vary significantly along pseudotime in time-series scRNA-seq data and a nonparametric null distribution is needed for a curve-vs-flat-line F statistic.

## When NOT to use

- Data lacks a trajectory/pseudotime ordering (scTITANS depends on pseudotime from Monocle3 trajectory inference to define the time-dependent covariate).
- The analysis goal is static two-condition differential expression without a time-series or pseudotime dimension — standard DE tools are more appropriate.
- Only raw, unprocessed data is available and only the bootstrap step is needed — scTITANS in 'full' mode would rerun preprocessing; use scTITANS_full instead of the partial/statistical stage.

## Inputs

- Pseudotime-ordered gene expression matrix (genes in rows, cells in columns) from upstream Monocle3 trajectory inference
- Cell metadata (row cells) with cluster/cell type annotations
- Gene metadata (row genes, header must contain gene_short_name)
- fdr level (e.g., 0.01)

## Outputs

- SigGenes.fdr0.01.txt — significant genes with q-values below the 0.01 FDR level
- SigClusters.fdr0.01.txt — significant cell subclusters with q-values below the 0.01 FDR level (or an error message if cluster identification fails)

## How to apply

For each gene, fit a complete model along pseudotime using a p-dimensional linear basis (basis dimension chosen by singular value decomposition) for the fitted mean expression, and fit a flat-line null model; compute the F statistic from the sum of squared residuals of both models (equations 3-4). Generate the null distribution of each F statistic by bootstrap: resample the residuals obtained under the alternative (complete) model fit, add them back to the null model fit, and form the per-gene p-value from the frequency that bootstrap null statistics exceed the observed statistic (equation 5). Convert p-values to q-values with the Storey-style estimator incorporating the pi0 proportion of non-differentially expressed genes (equation 6), calling genes significant at an FDR level of 0.01. Apply the identical F-statistic/bootstrap/q-value procedure to binned pseudotime cell counts per subcluster to call significant cell subclusters at fdr 0.01; write results to SigGenes.fdr0.01.txt and SigClusters.fdr0.01.txt (an error message is emitted if cluster identification fails).

## Related tools

- **scTITANS** (Implements the complete-versus-flat null model fitting, residual bootstrap, F-statistic evaluation, and Storey-style q-value estimation for genes and cell subclusters along pseudotime) — https://github.com/ZJUFanLab/scTITANS
- **Monocle3** (Upstream supplier of trajectory and per-cell pseudotime values that define the time-dependent covariate for the statistical test)

## Examples

```
result = scTITANS_partial(data_file,cell_meta,gene_meta,"none",0.01)
```

## Evaluation signals

- Both output files SigGenes.fdr0.01.txt and SigClusters.fdr0.01.txt exist after the run.
- Each listed gene and cluster entry carries a q-value below the 0.01 fdr level, i.e., the complete model differs significantly from the flat-line null.
- Per-gene p-values are bounded in [0,1] and reflect the bootstrap frequency that null F statistics exceed the observed statistic.
- The q-value computation includes the pi0 estimate of the proportion of non-differentially expressed genes (Storey-style estimator, equation 6).
- If cluster significance cannot be determined, an error message is emitted rather than an empty cluster file.

## Limitations

- No software version numbers for scTITANS, Monocle3, or scran are stated, and no changelog is available for the scTITANS repository.
- Significant cluster calling can fail (error message emitted instead of SigClusters.fdr0.01.txt).
- The basis dimension for the fitted mean is data-dependent (chosen by singular value decomposition), so results may vary with the fitted model complexity.

## Evidence

- [methods] For DEGs, a curve is fitted for the dynamic expression of each gene along the pseudotime: "For DEGs, a curve is fitted for the dynamic expression of each gene along the pseudotime"
- [methods] The null distribution of the statistic was calculated through a method named bootstrap: "The null distribution of the statistic was calculated through a method named bootstrap"
- [methods] scTITANS utilizes Monocle3 [13], [35] to construct the trajectory and calculate the pseudotime for each cell.: "scTITANS utilizes Monocle3 [13], [35] to construct the trajectory and calculate the pseudotime for each cell."
- [methods] a time-dependent covariate [37] was introduced to evaluate the fluctuations of genes and cell numbers along pseudotime: "a time-dependent covariate [37] was introduced to evaluate the fluctuations of genes and cell numbers along pseudotime"
- [readme] users chose no cell annotation and fdr level of 0.01 in identifying key genes and cell subclusters: "users chose no cell annotation and fdr level of 0.01 in identifying key genes and cell subclusters"
