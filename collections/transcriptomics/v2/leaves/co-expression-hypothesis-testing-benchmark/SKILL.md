---
name: co-expression-hypothesis-testing-benchmark
description: Use when evaluating or benchmarking scRNA-seq co-expression detection
  methods that output gene-pair p-values, by simulating UMI counts from a ground-truth
  sparse co-expression network and scoring precision-recall and runtime.
when_to_use_negative:
- The task is co-expression estimation accuracy or module/GO analysis rather than
  benchmarking detection accuracy and runtime of p-value-based methods
- The comparison involves methods without hypothesis tests (e.g. plain network-module
  tools), since this benchmark's PR analysis requires per-gene-pair p-values
- Input is real data with no ground-truth co-expression network, so precision-recall
  cannot be computed
edam_operation: http://edamontology.org/operation_2478
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2269
tools:
- name: CS-CORE (CSCORE R package)
  role: Primary method under benchmark; run via CSCORE IRLS/WLS on simulated counts;
    also used to estimate the ground-truth sparse co-expression matrix
  repo: https://github.com/ChangSuBiostats/CS-CORE
- name: locCSN
  role: Competing co-expression hypothesis-testing method applied to the simulated
    UMI counts
  repo: https://github.com/xuranw/locCSN
- name: normalisr
  role: Competing co-expression hypothesis-testing method applied to the simulated
    UMI counts
  repo: https://github.com/lingfeiwang/normalisr
- name: NoiseRegularization
  role: Competing co-expression hypothesis-testing method applied to the simulated
    UMI counts
  repo: https://github.com/RuoyuZhang/NoiseRegularization
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-40503-7
    title: Cell-type-specific co-expression inference from single cell RNA-sequencing
      data
  source_content_hash: sha256:41e6293da788f8cc52adc993cf89a52e30220b176711c15f05cbd283f7684f2b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ChangSuBiostats/CS-CORE
  tools:
  - CS-CORE
  - CS-CORE (CSCORE R package)
  - CSCORE (R package)
  - NoiseRegularization
  - locCSN
  - normalisr
  edam_operation: http://edamontology.org/operation_2478
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# co-expression-hypothesis-testing-benchmark

## Summary

Benchmark protocol comparing CS-CORE against co-expression methods with hypothesis tests (Pearson, Spearman, locCSN, normalisr, NoiseRegularization) on simulated UMI counts, using precision-recall detection accuracy and wall-clock running time as the evaluation axes (Fig. 3b,c of the CS-CORE paper).

## When to use

Use when evaluating or benchmarking scRNA-seq co-expression detection methods that output gene-pair p-values, by simulating UMI counts from a ground-truth sparse co-expression network and scoring precision-recall and runtime.

## When NOT to use

- The task is co-expression estimation accuracy or module/GO analysis rather than benchmarking detection accuracy and runtime of p-value-based methods
- The comparison involves methods without hypothesis tests (e.g. plain network-module tools), since this benchmark's PR analysis requires per-gene-pair p-values
- Input is real data with no ground-truth co-expression network, so precision-recall cannot be computed

## Inputs

- Ground-truth sparse co-expression matrix (p x p), e.g. CS-CORE-estimated from ref. 17 postmortem Alzheimer's disease scRNA-seq data
- Simulated UMI count matrix (p=500 genes x n=5000 cells) from the Poisson latent-variable measurement model with cell-specific size factors s_i
- Co-expression methods with hypothesis tests: CS-CORE (CSCORE IRLS/WLS), Pearson, Spearman, locCSN, normalisr, NoiseRegularization

## Outputs

- Precision-recall curves for all methods over the p x p gene pairs (Fig. 3b analogue)
- Table of per-method wall-clock running times (Fig. 3c analogue)
- Per-method BH-adjusted p-value matrices for all gene pairs

## How to apply

Start from a ground-truth sparse co-expression matrix, e.g. one estimated with CS-CORE (BH-adjusted p > 0.05 set to 0) from the ref. 17 postmortem Alzheimer's disease scRNA-seq dataset. Simulate UMI counts for p=500 genes and n=5000 cells under the latent-variable Poisson measurement model x_ij|z_ij ~ Poisson(s_i * z_ij), using the sparse matrix as the true network. Apply each competing method that provides a statistical test — CS-CORE (via CSCORE IRLS/WLS), Pearson, Spearman, locCSN, normalisr, NoiseRegularization — recording wall-clock time for each run on a single Xeon Gold-class node. Compute Benjamini-Hochberg adjusted p-values over all p x p gene-pair tests per method (e.g. p.adjust(method = 'BH')) and derive precision-recall curves over the gene pairs. Plot all PR curves together and tabulate per-method runtimes; validation is reproducing the Fig. 3b,c pattern in which CS-CORE achieves the highest AUC among methods with statistical tests while remaining computationally competitive.

## Related tools

- **CS-CORE (CSCORE R package)** (Primary method under benchmark; run via CSCORE IRLS/WLS on simulated counts; also used to estimate the ground-truth sparse co-expression matrix) — https://github.com/ChangSuBiostats/CS-CORE
- **locCSN** (Competing co-expression hypothesis-testing method applied to the simulated UMI counts) — https://github.com/xuranw/locCSN
- **normalisr** (Competing co-expression hypothesis-testing method applied to the simulated UMI counts) — https://github.com/lingfeiwang/normalisr
- **NoiseRegularization** (Competing co-expression hypothesis-testing method applied to the simulated UMI counts) — https://github.com/RuoyuZhang/NoiseRegularization

## Examples

```
CSCORE_result <- CSCORE(sim_counts, genes = genes_selected); p_bh <- p.adjust(CSCORE_result$p_matrix[upper.tri(CSCORE_result$p_matrix)], method = "BH")
```

## Evaluation signals

- CS-CORE achieves the highest area-under-the-curve value among methods with statistical tests in the p=500, n=5000 detection simulations
- Per-method wall-clock times show CS-CORE remains computationally competitive on a single Xeon Gold-class node
- Every method's p-values are BH-adjusted over all upper-triangular gene pairs before constructing the PR curve
- PR curves qualitatively reproduce the ordering of Fig. 3b across Pearson, Spearman, locCSN, normalisr, NoiseRegularization and CS-CORE
- Simulated counts match the Poisson measurement model x_ij|z_ij ~ Poisson(s_i * z_ij) with the specified sparse true network

## Limitations

- Benchmark validity depends on the ground-truth network itself being a CS-CORE estimate, which may favor CS-CORE's detection properties
- The measurement model uses only a Poisson distribution; adaptation to other count distributions such as negative binomial is deferred to future work
- CS-CORE assumes gene expressions from cells of the same cell type follow the same distribution, which may not hold across individuals with different genetic, demographic and clinical characteristics
- Runtimes are tied to a single Xeon Gold-class node and may not transfer to other hardware

## Evidence

- [other] CS-CORE achieved the highest area-under-the-curve value among methods with statistical tests in the p=500, n=5000 detection simulations.: [evidence span withheld — anchor status: not_found]
- [other] Poisson measurement model: [evidence span withheld — anchor status: not_found]
- [methods] BH-adjusted p-values: "p.adjust(CSCORE_p[upper.tri(CSCORE_p)], method = "BH")"
- [discussion] Poisson-only limitation: "It can be useful to adapt CS-CORE to model xij∣zij using other distributions that model nonnegative integers such as the negative binomial distribution and we leave it to future work."
- [readme] CS-CORE repo README: "`CS-CORE` is a R package for cell-type-specific co-expression inference from single cell RNA-sequencing data."
