---
name: wall-clock-timing-measurement
description: Use when comparing computational scalability of scRNAseq dimension reduction
  methods (e.g., CA via corral vs glmPCA) and you need reproducible per-dataset wall-clock
  timings on standardized count matrices.
when_to_use_negative:
- The goal is clustering accuracy or embedding quality (e.g., ARI evaluation) rather
  than computational scalability — the paper's benchmarking of performance against
  reference labels is a separate task
- Matrices have not been standardized to a common feature count (1500 features), since
  timing comparisons across differently sized inputs are not interpretable as a scalability
  result
- Comparing corralm multi-table batch integration rather than single-table dimension
  reduction, where the corral single-table call is the relevant entry point
tools:
- name: corral
  role: Standard CA implementation used in the timing test; interfaces directly with
    Bioconductor SingleCellExperiment classes and computes cell embeddings via an
    irlba-based partial SVD
  repo: github.com/laurenhsu1/corral
- name: glmPCA
  role: Baseline generalized-PCA method run on the same 1500-feature down-selected
    matrices to establish the contrast in wall-clock scaling behavior
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-022-26434-1
    title: Correspondence analysis for dimension reduction, batch integration, and
      visualization of single-cell RNA-seq data
  source_content_hash: sha256:77f1ad6cd82650b7e7b92374c87d125f2d6bc8eb1d981df33a15bbf30e83ebbe
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - R
  - corral
  - glmPCA
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# wall-clock-timing-measurement

## Summary

Benchmarks the computational scalability of dimension reduction methods for scRNAseq by recording wall-clock runtime — here, standard CA via corral versus glmPCA — on matched down-selected count matrices (1500 features), validating that CA scales to ~22,000-cell datasets in under a minute.

## When to use

Use when comparing computational scalability of scRNAseq dimension reduction methods (e.g., CA via corral vs glmPCA) and you need reproducible per-dataset wall-clock timings on standardized count matrices.

## When NOT to use

- The goal is clustering accuracy or embedding quality (e.g., ARI evaluation) rather than computational scalability — the paper's benchmarking of performance against reference labels is a separate task
- Matrices have not been standardized to a common feature count (1500 features), since timing comparisons across differently sized inputs are not interpretable as a scalability result
- Comparing corralm multi-table batch integration rather than single-table dimension reduction, where the corral single-table call is the relevant entry point

## Inputs

- Ten benchmark single-cell count datasets (Zhengmix4eq, SCMixology, and related benchmarks) with original-publication cell labels
- Down-selected count matrices standardized to 1500 features per dataset
- R/Bioconductor environment with corral and glmPCA installed

## Outputs

- runtime_comparison.csv: table of elapsed wall-clock time per method (corral CA, glmPCA) per dataset
- Per-dataset runtime records distinguishing sub-minute (CA) from multi-hour (glmPCA) runs on ~22,000-cell inputs

## How to apply

Load the ten benchmark single-cell count datasets (Zhengmix4eq, SCMixology, and related benchmarks retaining original-publication cell labels), and down-select each to 1500 features so both methods see an identically sized input matrix. Run standard CA with the corral R/Bioconductor package — which uses an irlba-based partial SVD — wrapping the call in a wall-clock timer (e.g., system.time), then run glmPCA on the same down-selected matrices and record its elapsed time the same way. The timing decision point is matrix size: compare methods at equal feature counts rather than on the full feature space, since scalability is the claim under test. Compile per-dataset, per-method runtimes into a single table runtime_comparison.csv. The rationale comes from the paper's scalability finding: corral's irlba-based partial SVD is 'fast, scalable', and should complete in under a minute even for datasets with over 20,000 cells, while glmPCA requires over an hour for the equivalent 1500-feature x ~22,000-cell input.

## Related tools

- **corral** (Standard CA implementation used in the timing test; interfaces directly with Bioconductor SingleCellExperiment classes and computes cell embeddings via an irlba-based partial SVD) — github.com/laurenhsu1/corral
- **glmPCA** (Baseline generalized-PCA method run on the same 1500-feature down-selected matrices to establish the contrast in wall-clock scaling behavior)

## Examples

```
system.time(corral(sce))  # wall-clock timing of standard CA on a SingleCellExperiment down-selected to 1500 features
```

## Evaluation signals

- Standard CA (corral) runtime is under one minute for every benchmark dataset, including those with over 20,000 cells
- glmPCA runtime exceeds one hour on the equivalent 1500-feature x ~22,000-cell input matrix
- runtime_comparison.csv contains one runtime row per method per dataset across all ten benchmarks
- Both methods were run on the identical down-selected matrix (1500 features) for each dataset, so timings are directly comparable
- Timings reflect wall-clock elapsed time as recorded with a timer around the full method call (data loading excluded)

## Limitations

- No tool or package versions for corral or its dependencies are reported, so exact runtimes may not be reproducible across versions
- Reported timing behavior (CA under a minute vs glmPCA over an hour) is specific to the 1500-feature down-selected matrices and ~22,000-cell datasets; it does not generalize to other matrix sizes or hardware
- Runtime is only one axis of the paper's benchmark; it does not address clustering accuracy, and the paper notes ARI-based comparisons may penalize methods that better distinguish sub-populations

## Evidence

- [abstract] CA scales favorably relative to glmPCA: "we propose five adaptations of CA, which are fast, scalable, and outperform standard CA and glmPCA"
- [intro] corral is the Bioconductor CA implementation used for the timing run: "We implemented standard and adapted CA for scRNAseq in corral, an R/Bioconductor package that interfaces directly with Bioconductor classes"
- [readme] corral single-table call is the timed entry point: "1. `corral` for dimensionality reduction on a single table"
- [readme] corral targets scRNAseq count data as timed here: "corral is a package to perform correspondence analysis on count data, particularly for single-cell (e.g., RNA-seq)."
