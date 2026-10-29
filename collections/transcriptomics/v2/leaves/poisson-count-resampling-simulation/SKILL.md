---
name: poisson-count-resampling-simulation
description: 'Use when benchmarking type-I error control of scRNA-seq co-expression
  methods: you need null count matrices where genes are not co-expressed, in both
  constant-depth and depth-confounded variants.'
when_to_use_negative:
- The goal is estimating real co-expression from observed data rather than simulating
  null data — use CS-CORE/CSCORE directly on observed counts instead
- The null must preserve gene-gene dependence structure (e.g. for power or module-recovery
  simulations); permutation destroys co-expression by design
- Input is normalized/log-transformed expression rather than raw UMI integer counts,
  since the Poisson resampling model operates on raw counts and depths t_i
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CSCORE (R package)
  role: 'Benchmark target: the null count matrices are used to evaluate CS-CORE''s
    type-I error control under constant and varying sequencing depth'
  repo: github.com/ChangSuBiostats/CS-CORE
- name: stats
  role: Base R functions for per-gene permutation of normalized expression and Poisson
    count resampling
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-40503-7
    title: Cell-type-specific co-expression inference from single cell RNA-sequencing
      data
  source_content_hash: sha256:bd5991fa1d4e4b18625369d443721350c2ec274ae152dc138391a5ff38d088be
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CS-CORE
  - CSCORE (R package)
  - stats
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# poisson-count-resampling-simulation

## Summary

Generates null scRNA-seq UMI count matrices with no true gene co-expression by per-cell sequencing-depth normalization, per-gene permutation of normalized expression across cells, and Poisson resampling of counts under either constant or observed (varying) sequencing depths. This provides the null data needed to benchmark false-positive rates of co-expression methods such as CS-CORE.

## When to use

Use when benchmarking type-I error control of scRNA-seq co-expression methods: you need null count matrices where genes are not co-expressed, in both constant-depth and depth-confounded variants.

## When NOT to use

- The goal is estimating real co-expression from observed data rather than simulating null data — use CS-CORE/CSCORE directly on observed counts instead
- The null must preserve gene-gene dependence structure (e.g. for power or module-recovery simulations); permutation destroys co-expression by design
- Input is normalized/log-transformed expression rather than raw UMI integer counts, since the Poisson resampling model operates on raw counts and depths t_i

## Inputs

- Raw UMI count matrix from a scRNA-seq dataset (e.g. GSE157827, excitatory-neuron cells subset)
- Per-cell sequencing depth vector t_i (column sums of the count matrix)

## Outputs

- Constant-depth null count matrix: permuted normalized expressions resampled with equal depths via x*_ij ~ Poisson(t_i * y*_ij^p)
- Varying-depth null count matrix: Poisson resampling retaining observed cell-specific depths t_i
- Saved R object file null_count_matrices.rds containing both matrices

## How to apply

Start from a raw UMI count matrix (e.g. GSE157827 restricted to excitatory-neuron cells) and normalize each cell's counts by its sequencing depth t_i to obtain y_ij = x_ij / t_i. For each gene independently, permute its normalized expression values across cells so gene-gene co-expression is destroyed while each gene's marginal distribution is preserved. Then resample null counts from the Poisson measurement model x*_ij ~ Poisson(t_i * y*_ij^p): with equal depths t_i across cells to produce the constant-depth null matrix (no depth confounding), and with the observed cell-specific depths retained to produce the varying-depth null matrix that mimics the depth confounding CS-CORE models. Save both matrices (e.g. to null_count_matrices.rds) and validate before use.

## Related tools

- **CSCORE (R package)** (Benchmark target: the null count matrices are used to evaluate CS-CORE's type-I error control under constant and varying sequencing depth) — github.com/ChangSuBiostats/CS-CORE
- **stats** (Base R functions for per-gene permutation of normalized expression and Poisson count resampling)

## Examples

```
y <- t(t(x)/t_i); y_star <- apply(y, 2, sample); x_star_vary <- matrix(rpois(length(y_star), as.vector(t_i %o% colSums(x)/sum(t_i) * y_star)), nrow(y)); saveRDS(x_star_vary, 'null_count_matrices.rds')
```

## Evaluation signals

- Both null matrices contain only nonnegative integer counts (Poisson resampling output check)
- Each gene's marginal expression distribution is preserved relative to the observed data after permutation
- Permutation yields near-zero co-expression between gene pairs, as required by the null hypothesis for type-I error benchmarking
- The varying-depth null matrix reproduces the observed per-cell sequencing depths, while the constant-depth matrix uses equal depths across cells

## Limitations

- The measurement model uses only a Poisson distribution; adaptation to other count distributions such as the negative binomial is deferred to future work
- Permutation preserves marginal gene distributions but cannot simulate nulls with realistic correlation structure other than zero
- The varying-depth null intentionally embeds depth confounding, so false positives on it do not indicate a method flaw unless the method claims depth robustness

## Evidence

- [methods] Permutation destroys co-expression while preserving marginals: [evidence span withheld — anchor status: not_found]
- [intro] Depth confounding motivates the varying-depth null: "gene co-expressions measured via correlations of UMI counts across cells can be seriously confounded by varying sequencing depths, resulting in inflated false positive findings"
- [abstract] Poisson measurement model is the generative basis: [evidence span withheld — anchor status: not_found]
- [discussion] Poisson-only count model limitation: "It can be useful to adapt CS-CORE to model xij∣zij using other distributions that model nonnegative integers such as the negative binomial distribution"
