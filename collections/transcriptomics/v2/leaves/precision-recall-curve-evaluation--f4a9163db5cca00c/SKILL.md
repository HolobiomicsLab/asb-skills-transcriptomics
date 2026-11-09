---
name: precision-recall-curve-evaluation
description: Use when comparing gene co-expression detection methods (e.g., CS-CORE,
  Pearson, Spearman, locCSN, normalisr, NoiseRegularization) that output p-values,
  using a known ground-truth sparse co-expression network as the reference.
when_to_use_negative:
- A candidate method produces only point co-expression estimates without p-values
  or test statistics, so no BH-adjusted ranked gene pairs can be derived for a PR
  curve.
- No ground-truth co-expression network is available (e.g., real data without a known
  network), making precision-recall evaluation impossible.
- The goal is co-expression value accuracy or module detection rather than binary
  detection accuracy with FDR control.
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0092
tools:
- name: CSCORE (R package)
  role: Runs the CS-CORE hypothesis test (CSCORE IRLS/WLS) on simulated counts to
    produce p-values for the precision-recall comparison
  repo: https://github.com/ChangSuBiostats/CS-CORE
- name: locCSN
  role: Competing co-expression method with statistical tests whose p-values are ranked
    in the PR benchmark
  repo: https://github.com/xuranw/locCSN
- name: normalisr
  role: Competing normalization and co-expression association-testing framework included
    in the PR comparison
  repo: https://github.com/lingfeiwang/normalisr
- name: NoiseRegularization
  role: Competing preprocessing/regularization method included in the PR comparison
    of detection accuracy
  repo: https://github.com/RuoyuZhang/NoiseRegularization
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-40503-7
    title: Cell-type-specific co-expression inference from single cell RNA-sequencing
      data
  source_content_hash: sha256:e0ffd3621cf79686f737ca781eb01fc7ad8b2b29be6abb03927fae7987b73b1a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ChangSuBiostats/CS-CORE
  tools:
  - CS-CORE
  - CSCORE (R package)
  - NoiseRegularization
  - locCSN
  - normalisr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# precision-recall-curve-evaluation

## Summary

Benchmark protocol from the CS-CORE paper for comparing co-expression detection accuracy among methods with hypothesis tests, by simulating UMI counts (Poisson measurement model) and computing precision-recall curves from Benjamini-Hochberg adjusted p-values over all gene pairs.

## When to use

Use when comparing gene co-expression detection methods (e.g., CS-CORE, Pearson, Spearman, locCSN, normalisr, NoiseRegularization) that output p-values, using a known ground-truth sparse co-expression network as the reference.

## When NOT to use

- A candidate method produces only point co-expression estimates without p-values or test statistics, so no BH-adjusted ranked gene pairs can be derived for a PR curve.
- No ground-truth co-expression network is available (e.g., real data without a known network), making precision-recall evaluation impossible.
- The goal is co-expression value accuracy or module detection rather than binary detection accuracy with FDR control.

## Inputs

- Ground-truth sparse co-expression matrix (p x p) used as the simulated true network
- Simulated UMI count matrix for p=500 genes x n=5000 cells from the Poisson measurement model with size factors s_i
- Per-method p-value matrices over gene pairs (upper triangular) and wall-clock running times

## Outputs

- Precision-recall curves for each hypothesis-test-based method over the p x p gene pairs
- Per-method area-under-the-curve values
- Table of per-method wall-clock running times on a single Xeon Gold-class node

## How to apply

Load a ground-truth sparse co-expression matrix (in the paper, estimated with CS-CORE from the ref. 17 postmortem Alzheimer's disease scRNA-seq dataset) and simulate UMI counts for p=500 genes and n=5000 cells under the latent-variable Poisson model x_ij|z_ij ~ Poisson(s_i * z_ij). Run each method with a hypothesis test (CSCORE IRLS/WLS, Pearson, Spearman, locCSN, normalisr, NoiseRegularization) on the same simulated counts, recording wall-clock running time per method. BH-adjust p-values with p.adjust(..., method = "BH") over the upper-triangular gene pairs and rank all p(p-1)/2 pairs to build a precision-recall curve per method. Compare methods by area-under-the-curve and report per-method running times in a table. The expected outcome is that CS-CORE attains the highest AUC among test-based methods while remaining computationally competitive.

## Related tools

- **CSCORE (R package)** (Runs the CS-CORE hypothesis test (CSCORE IRLS/WLS) on simulated counts to produce p-values for the precision-recall comparison) — https://github.com/ChangSuBiostats/CS-CORE
- **locCSN** (Competing co-expression method with statistical tests whose p-values are ranked in the PR benchmark) — https://github.com/xuranw/locCSN
- **normalisr** (Competing normalization and co-expression association-testing framework included in the PR comparison) — https://github.com/lingfeiwang/normalisr
- **NoiseRegularization** (Competing preprocessing/regularization method included in the PR comparison of detection accuracy) — https://github.com/RuoyuZhang/NoiseRegularization

## Examples

```
p_matrix_BH <- p.adjust(CSCORE_p[upper.tri(CSCORE_p)], method = "BH")
```

## Evaluation signals

- CS-CORE attains the highest area-under-the-curve value among methods with statistical tests, reproducing the qualitative pattern of Fig. 3b,c
- Every method's p-values are BH-adjusted (p.adjust method = "BH") and PR curves are derived over the full p x p gene pairs
- Wall-clock running time is recorded for every method on a single Xeon Gold-class node and CS-CORE remains computationally competitive
- The simulation uses the latent-variable Poisson measurement model x_ij|z_ij ~ Poisson(s_i * z_ij) with the sparse ground-truth co-expression matrix at p=500 genes, n=5000 cells

## Limitations

- The measurement model uses only a Poisson distribution; adaptation to other count distributions (e.g., negative binomial) is deferred to future work, so benchmark conclusions may not transfer to overdispersed counts.
- CS-CORE assumes gene expressions from cells of the same cell type follow the same distribution, which may not hold across individuals with different genetic, demographic and clinical characteristics.
- Running-time results are hardware-dependent (measured on a single Xeon Gold-class node) and thus not directly comparable across machines.

## Evidence

- [methods] Precision-recall construction from BH-adjusted p-values: "p.adjust(CSCORE_p[upper.tri(CSCORE_p)], method = "BH")"
- [abstract] Inflated false positives motivate detection-accuracy benchmarking: "most existing methods suffered from inflated false positives as well as biased co-expression estimates and clustering analysis"
- [intro] Existing methods' type-I error and power deficiencies: [evidence span withheld — anchor status: not_found]
- [readme] CS-CORE package used to generate test p-values for the benchmark: "`CS-CORE` is a R package for cell-type-specific co-expression inference from single cell RNA-sequencing data."
- [methods] CSCORE call producing co-expression estimates and p-values: "CSCORE_result <- CSCORE(pbmc_B_healthy, genes = genes_selected)"
