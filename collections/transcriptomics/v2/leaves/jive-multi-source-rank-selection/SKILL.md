---
name: jive-multi-source-rank-selection
description: Use when you have multiple matched data matrices (e.g. BRCA gene expression,
  methylation, miRNA) and must decide the joint rank and per-source individual ranks
  before interpreting a JIVE decomposition.
when_to_use_negative:
- Data are large-scale single-cell scRNA-seq batches — the original r.jive implementation
  was designed for bulk sequencing data and is computationally infeasible there; use
  scJIVE instead
- Runtime and peak memory are tightly constrained and a Python AJIVE (mvdr) implementation
  is acceptable — AJIVE fitted the same scenario in ~18 s / 601 MB vs ~2 h / 1180
  MB for jive()
- Data sources are not matched across samples/rows, since JIVE requires jointly decomposable
  multi-source matrices
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0203
tools:
- name: r.jive
  role: R package providing jive(), jive.perm() and bic.jive() for fitting the JIVE
    decomposition and selecting ranks by permutation or BIC
- name: R
  role: Runtime environment; package installed via install.packages("r.jive")
- name: SpatioTemporal
  role: Dependency used for imputing missing values via SVDmiss before decomposition
- name: gplots
  role: Dependency used for plotting functions such as showHeatmaps
- name: AJIVE (Python mvdr library)
  role: Alternative joint-rank estimator used as a faster, more memory-efficient baseline
    in the runtime/memory comparison
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioadv/vbae134
    title: Batch-effect correction in single-cell RNA sequencing data using JIVE
  source_content_hash: sha256:cb2d86c34c169aa79f7896b2d0a8d2efafd50ccc7f6fb60bbe4907a1816a1c89
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AJIVE (Python mvdr library)
  - R
  - SpatioTemporal
  - gplots
  - r.jive
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# jive-multi-source-rank-selection

## Summary

Selects the joint and individual ranks for a JIVE decomposition of multi-source data using r.jive's two built-in approaches: a permutation-based test (jive.perm) described by Lock et al. and a BIC-motivated criterion (bic.jive). Correct rank selection is what determines how much variation is attributed to shared vs. batch-specific structure.

## When to use

Use when you have multiple matched data matrices (e.g. BRCA gene expression, methylation, miRNA) and must decide the joint rank and per-source individual ranks before interpreting a JIVE decomposition.

## When NOT to use

- Data are large-scale single-cell scRNA-seq batches — the original r.jive implementation was designed for bulk sequencing data and is computationally infeasible there; use scJIVE instead
- Runtime and peak memory are tightly constrained and a Python AJIVE (mvdr) implementation is acceptable — AJIVE fitted the same scenario in ~18 s / 601 MB vs ~2 h / 1180 MB for jive()
- Data sources are not matched across samples/rows, since JIVE requires jointly decomposable multi-source matrices

## Inputs

- Multiple matched feature-by-sample matrices (e.g. BRCA_data: gene expression, DNA methylation, miRNA from TCGA breast cancer samples)
- Simulated multi-source data (SimData from the r.jive package)
- Rank-selection settings: number of permutations (nperms), significance level (alpha), convergence tolerance (conv), max iterations (maxiter), cores for parallelism

## Outputs

- A jive output object containing the joint low-rank estimate and per-source individual low-rank estimates with row-orthogonality enforced
- Selected joint rank and per-source individual ranks (e.g. joint rank 2; individual ranks 27, 26, 25 on BRCA_data)
- Diagnostic plots: showVarExplained barchart, showHeatmaps, showPCA point-cloud view

## How to apply

Load the matched data sources (r.jive ships example datasets BRCA_data and SimData) and pass them to the r.jive package in R; the default estimation function is `jive`. For rank selection, choose between the permutation-based approach (`jive.perm(data, nperms, alpha, est, conv, maxiter, orthIndiv, showProgress, cores)`) and the BIC-motivated approach (`bic.jive(data, n, d, conv, maxiter, orthIndiv, showProgress, cores)`), each of which itself calls the underlying `jive` estimator with row-orthogonality enforced between joint and individual estimates. After fitting, verify the selected ranks with `showVarExplained` (barchart of variation explained), `showHeatmaps`, or `showPCA`, and inspect the final joint and individual ranks in the jive output. As a sanity benchmark, on the packaged BRCA_data the permutation approach estimated a final joint rank of 2 with individual ranks of 27 (gene expression), 26 (methylation), and 25 (miRNA), and the joint structure corresponded to three previously identified sample clusters. Note that permutation-based selection is computationally expensive: in a head-to-head runtime/memory comparison, r.jive's jive() took slightly over 2 h and ~1180 MB versus ~18 s and ~601 MB for the Python mvdr AJIVE implementation on the same simulation scenario.

## Related tools

- **r.jive** (R package providing jive(), jive.perm() and bic.jive() for fitting the JIVE decomposition and selecting ranks by permutation or BIC)
- **R** (Runtime environment; package installed via install.packages("r.jive"))
- **SpatioTemporal** (Dependency used for imputing missing values via SVDmiss before decomposition)
- **gplots** (Dependency used for plotting functions such as showHeatmaps)
- **AJIVE (Python mvdr library)** (Alternative joint-rank estimator used as a faster, more memory-efficient baseline in the runtime/memory comparison)

## Examples

```
install.packages("r.jive"); library(r.jive); data(BRCA_data); result <- jive(BRCA_data); showVarExplained(result)
```

## Evaluation signals

- jive output reports a final joint rank and individual ranks; on packaged BRCA_data the expected outcome is joint rank 2 with individual ranks 27, 26, 25
- The estimated joint structure reproduces three previously identified BRCA sample clusters (loaded as the vector clusts)
- Row-orthogonality is enforced between joint and individual estimates and between each individual estimate
- showVarExplained barchart and showHeatmaps render without error and partition variation across joint/individual components
- In benchmark settings, a record with runtime_s, memory_mb, and selected_joint_rank fields exists; a selected joint rank of 0 matches the AJIVE-vs-JIVE discussion comparison (~18 s vs ~2 h; ~601 MB vs ~1180 MB)

## Limitations

- JIVE's original r.jive implementation was designed for bulk sequencing data and is computationally infeasible for large-scale single-cell datasets
- Permutation-based rank selection (jive.perm) is expensive; on a simulation scenario it took slightly over 2 h and 1180 MB versus 18 s and 601 MB for Python mvdr AJIVE
- Two rank-selection philosophies exist (permutation-based and BIC-motivated) and they can yield different rank estimates; the choice must be reported alongside the decomposition

## Evidence

- [methods] permutation vs BIC rank selection: "One is a permutation-based approach described in @Lock, the other is a BIC-motivated approach."
- [methods] BRCA rank outcome: "Final joint rank: 2 , final individual ranks: 27 26 25"
- [methods] clusters match: "We see that the estimated joint corresponds well to the three previously identified clusters"
- [methods] orthogonality: "with row-orthogonality enforced between the joint and invidual estimates and also between each individual estimate"
- [abstract] computational infeasibility for scRNA-seq: "its current implementation is originally designed for bulk sequencing data, making it computationally infeasible for large-scale single-cell sequencing datasets."
- [methods] r.jive vignette usage: "This vignette illustrates the usage and capabilities of the **r.jive** package"
