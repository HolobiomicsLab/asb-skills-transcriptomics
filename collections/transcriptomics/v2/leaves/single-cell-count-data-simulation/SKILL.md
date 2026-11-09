---
name: single-cell-count-data-simulation
description: Use when benchmarking co-expression inference methods (e.g., CS-CORE
  vs Pearson, Spearman, locCSN, normalisr, NoiseRegularization) against a known true
  network, or when evaluating whether estimation bias harms downstream clustering/PCA
  via simulations with controlled n, p, and cluster structure.
when_to_use_negative:
- Analyzing a real scRNA-seq dataset where no ground-truth co-expression network exists
  — simulation is a benchmarking device, not an analysis pipeline step
- You require realistic overdispersion in the count-generating distribution — the
  model here is Poisson-only; the authors note negative binomial adaptation is deferred
  to future work
- The question concerns differential expression or cell-type annotation rather than
  co-expression estimation accuracy
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CS-CORE / CSCORE (R package)
  role: Benchmarked method (CSCORE IRLS/WLS) and source of the sparse ground-truth
    co-expression matrix used to seed simulations
  repo: github.com/ChangSuBiostats/CS-CORE
- name: locCSN
  role: Comparison co-expression method with hypothesis tests applied to simulated
    counts
  repo: github.com/xuranw/locCSN
- name: normalisr
  role: Comparison co-expression method with hypothesis tests applied to simulated
    counts
  repo: github.com/lingfeiwang/normalisr
- name: NoiseRegularization
  role: Comparison preprocessing/co-expression method applied to simulated counts
  repo: github.com/RuoyuZhang/NoiseRegularization
provenance:
  source_task_ids:
  - task_006
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-40503-7
    title: Cell-type-specific co-expression inference from single cell RNA-sequencing
      data
  source_content_hash: sha256:b3bebeb23cb8897684c0355cfac20a4136628324c7c80e85ff74c3801b02d420
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CS-CORE
  - CS-CORE / CSCORE (R package)
  - CSCORE (R package)
  - NoiseRegularization
  - locCSN
  - normalisr
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# single-cell-count-data-simulation

## Summary

Generates synthetic scRNA-seq UMI count data with known ground-truth co-expression structure using a latent-variable Poisson measurement model, enabling controlled benchmarks of co-expression estimation, clustering, and PCA recovery methods.

## When to use

Use when benchmarking co-expression inference methods (e.g., CS-CORE vs Pearson, Spearman, locCSN, normalisr, NoiseRegularization) against a known true network, or when evaluating whether estimation bias harms downstream clustering/PCA via simulations with controlled n, p, and cluster structure.

## When NOT to use

- Analyzing a real scRNA-seq dataset where no ground-truth co-expression network exists — simulation is a benchmarking device, not an analysis pipeline step
- You require realistic overdispersion in the count-generating distribution — the model here is Poisson-only; the authors note negative binomial adaptation is deferred to future work
- The question concerns differential expression or cell-type annotation rather than co-expression estimation accuracy

## Inputs

- Ground-truth co-expression matrix (sparse, e.g., CS-CORE-estimated from ref. 17 postmortem Alzheimer's disease scRNA-seq data) or a designed latent cluster structure
- Simulation design parameters: number of cells n (e.g., 2000 or 5000), number of genes p (e.g., 100 or 500), number and correlation of co-expression clusters
- Per-cell sequencing depth vector s_i with varying depths and measurement error

## Outputs

- Simulated UMI count matrix (n cells x p genes) with paired latent true expression z
- Ground-truth co-expression matrix and true gene-cluster/PCA labels
- Per-method evaluation metrics table (e.g., figure4_simulation_benchmark.csv with ARI, subspace distance, estimate correlations)
- Precision-recall curves and per-method running times for detection benchmarks

## How to apply

First fix the ground truth: either define latent true expression for n cells and p genes forming correlated co-expression clusters (e.g., four clusters as in the Fig. 4 simulations with n=2000, p=100), or adopt a sparse co-expression matrix estimated from real data (e.g., the CS-CORE network from the ref. 17 postmortem Alzheimer's disease scRNA-seq dataset) as in the p=500, n=5000 detection simulations. Then generate observed UMI counts through the Poisson measurement model x_ij|z_ij ~ Poisson(s_i * z_ij), where s_i encodes varying sequencing depths and measurement error across cells. Apply each candidate method to the simulated counts — CS-CORE via CSCORE IRLS/WLS plus Pearson, Spearman, locCSN, normalisr, and NoiseRegularization — recording wall-clock time per method. For detection benchmarks, compute Benjamini-Hochberg adjusted p-values for all gene-pair tests and derive precision-recall curves over the p x p pairs. For downstream-evaluation benchmarks, score methods by adjusted Rand index of co-expression cluster recovery, subspace distance between top four singular vectors of estimated vs true co-expression matrices, and correlation of estimated vs true mean expression and co-expression values, assembling results into a per-method metrics table.

## Related tools

- **CS-CORE / CSCORE (R package)** (Benchmarked method (CSCORE IRLS/WLS) and source of the sparse ground-truth co-expression matrix used to seed simulations) — github.com/ChangSuBiostats/CS-CORE
- **locCSN** (Comparison co-expression method with hypothesis tests applied to simulated counts) — github.com/xuranw/locCSN
- **normalisr** (Comparison co-expression method with hypothesis tests applied to simulated counts) — github.com/lingfeiwang/normalisr
- **NoiseRegularization** (Comparison preprocessing/co-expression method applied to simulated counts) — github.com/RuoyuZhang/NoiseRegularization

## Examples

```
counts <- sapply(seq_len(n), function(i) rpois(p, lambda = s[i] * z[, i]))  # x_ij|z_ij ~ Poisson(s_i * z_ij)
```

## Evaluation signals

- Precision-recall AUC over BH-adjusted (FDR <= 0.05) gene-pair tests: CS-CORE should achieve the highest AUC among test-based methods in the p=500, n=5000 setting
- Adjusted Rand index of recovered co-expression clusters: CS-CORE should be the only method accurately recovering four truly co-expressed gene clusters at n=2000, p=100
- Subspace distance between the top four singular vectors of estimated and true co-expression matrices should be minimal for CS-CORE and large for biased competitors
- Correlation of estimated vs true mean expression and co-expression values should approach 1 for CS-CORE, with inflated false positives (type-I error) visible for existing methods under null gene pairs
- Per-method wall-clock times should be recorded on a single Xeon Gold-class node to confirm CS-CORE remains computationally competitive

## Limitations

- The measurement model uses only a Poisson distribution; adaptation to other count distributions such as the negative binomial is explicitly deferred to future work
- The latent-variable model assumes gene expressions from cells of the same cell type follow the same distribution, which may not hold across individuals with different genetic, demographic, and clinical characteristics
- Detection-simulation ground truth inherits the sparsity pattern of a CS-CORE-estimated network from ref. 17, so benchmark conclusions are conditional on that estimated network being a reasonable truth
- Running-time comparisons are hardware-dependent (single Xeon Gold-class node in the reported benchmark)

## Evidence

- [results] Poisson measurement model: [evidence span withheld — anchor status: not_found]
- [results] Fig. 4 simulation design: [evidence span withheld — anchor status: not_found]
- [results] Depth and measurement error variation: [evidence span withheld — anchor status: not_found]
- [results] BH-adjusted PR evaluation: [evidence span withheld — anchor status: not_found]
- [discussion] Poisson-only limitation: "It can be useful to adapt CS-CORE to model xij∣zij using other distributions that model nonnegative integers such as the negative binomial distribution"
- [abstract] Rationale for the measurement model: [evidence span withheld — anchor status: not_found]
