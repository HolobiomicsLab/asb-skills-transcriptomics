---
name: poisson-measurement-model-simulation
description: Use when benchmarking co-expression detection methods on simulated scRNA-seq
  data that must preserve real co-expression structure — e.g., generating p x n UMI
  count matrices from a known sparse co-expression matrix to compute precision-recall
  curves and wall-clock runtimes per method.
when_to_use_negative:
- The input is observed real scRNA-seq data and the goal is co-expression inference
  rather than method benchmarking — run CS-CORE directly on the data instead of simulating.
- The count model must differ from Poisson (e.g., negative binomial) — the article
  explicitly defers adaptation of x_ij|z_ij to other distributions such as the negative
  binomial to future work.
- Input is normalized/transformed expression (not raw UMI counts) — the measurement
  model is defined on nonnegative-integer UMI counts confounded by sequencing depth
  and measurement error.
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CS-CORE / CSCORE (R package)
  role: Estimates the ground-truth sparse co-expression network and is benchmarked
    under its own latent-variable Poisson simulation model (IRLS/WLS implementations);
    supplies BH-adjusted p-values for the PR analysis.
  repo: https://github.com/ChangSuBiostats/CS-CORE
- name: locCSN
  role: Competing co-expression hypothesis-testing method applied to the simulated
    counts for precision-recall and runtime comparison.
  repo: https://github.com/xuranw/locCSN
- name: normalisr
  role: Competing normalization and association-testing method applied to the simulated
    counts for precision-recall and runtime comparison.
  repo: https://github.com/lingfeiwang/normalisr
- name: NoiseRegularization
  role: Competing preprocessing-based method applied to the simulated counts for precision-recall
    and runtime comparison.
  repo: https://github.com/RuoyuZhang/NoiseRegularization
- name: Pearson / Spearman correlation
  role: Baseline correlation-based hypothesis tests on the simulated counts, included
    in the PR-curve comparison.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-40503-7
    title: Cell-type-specific co-expression inference from single cell RNA-sequencing
      data
  source_content_hash: sha256:e3c4fc99fb06f788a27660d76a477c14c63ad78835ff3295d62e5838fdc6361b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ChangSuBiostats/CS-CORE
  tools:
  - CS-CORE
  - CS-CORE / CSCORE (R package)
  - CSCORE (R package)
  - NoiseRegularization
  - Pearson / Spearman correlation
  - locCSN
  - normalisr
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9523809523809523
n_claims: 21
---

# poisson-measurement-model-simulation

## Summary

Simulates synthetic scRNA-seq UMI count data under the CS-CORE latent-variable Poisson measurement model x_ij|z_ij ~ Poisson(s_i * z_ij), using a ground-truth sparse co-expression network estimated from real data, to benchmark co-expression methods via precision-recall and running-time comparisons.

## When to use

Use when benchmarking co-expression detection methods on simulated scRNA-seq data that must preserve real co-expression structure — e.g., generating p x n UMI count matrices from a known sparse co-expression matrix to compute precision-recall curves and wall-clock runtimes per method.

## When NOT to use

- The input is observed real scRNA-seq data and the goal is co-expression inference rather than method benchmarking — run CS-CORE directly on the data instead of simulating.
- The count model must differ from Poisson (e.g., negative binomial) — the article explicitly defers adaptation of x_ij|z_ij to other distributions such as the negative binomial to future work.
- Input is normalized/transformed expression (not raw UMI counts) — the measurement model is defined on nonnegative-integer UMI counts confounded by sequencing depth and measurement error.

## Inputs

- Ground-truth sparse co-expression matrix (p x p) estimated from a real scRNA-seq dataset (e.g., CS-CORE on the postmortem Alzheimer's disease dataset, ref. 17)
- Simulation dimensions and size factors (p=500 genes, n=5000 cells, cell-specific sequencing depths s_i)
- Simulated UMI count matrices consumed by the tested co-expression methods

## Outputs

- Simulated UMI count matrix x (n cells x p genes) under the Poisson measurement model
- Per-method BH-adjusted p-value matrices over all p x p gene pairs
- Precision-recall curves comparing detection accuracy across methods
- Per-method wall-clock running-time table

## How to apply

First obtain a ground-truth sparse co-expression matrix, e.g., estimated with CS-CORE (BH-adjusted p > 0.05 co-expressions set to 0) from a real scRNA-seq dataset such as the postmortem Alzheimer's disease brain dataset of ref. 17. Simulate latent true expression levels z_ij and draw observed UMI counts x_ij ~ Poisson(s_i * z_ij) with cell-specific size factors s_i, for the chosen dimensions (e.g., p=500 genes, n=5000 cells) so the sparse matrix is the true network. Apply every method with a hypothesis test (CS-CORE via CSCORE IRLS/WLS, Pearson, Spearman, locCSN, normalisr, NoiseRegularization) to the same simulated counts, recording wall-clock time for each. Compute Benjamini-Hochberg adjusted p-values for all p x p gene-pair tests per method and derive precision-recall curves over all gene pairs. Validate by reproducing the qualitative pattern of Fig. 3b,c: CS-CORE attains the highest area-under-the-curve among methods with statistical tests while remaining computationally competitive on a single Xeon Gold-class node.

## Related tools

- **CS-CORE / CSCORE (R package)** (Estimates the ground-truth sparse co-expression network and is benchmarked under its own latent-variable Poisson simulation model (IRLS/WLS implementations); supplies BH-adjusted p-values for the PR analysis.) — https://github.com/ChangSuBiostats/CS-CORE
- **locCSN** (Competing co-expression hypothesis-testing method applied to the simulated counts for precision-recall and runtime comparison.) — https://github.com/xuranw/locCSN
- **normalisr** (Competing normalization and association-testing method applied to the simulated counts for precision-recall and runtime comparison.) — https://github.com/lingfeiwang/normalisr
- **NoiseRegularization** (Competing preprocessing-based method applied to the simulated counts for precision-recall and runtime comparison.) — https://github.com/RuoyuZhang/NoiseRegularization
- **Pearson / Spearman correlation** (Baseline correlation-based hypothesis tests on the simulated counts, included in the PR-curve comparison.)

## Examples

```
library(devtools); install_github("ChangSuBiostats/CS-CORE"); CSCORE_result <- CSCORE(pbmc_B_healthy, genes = genes_selected)
```

## Evaluation signals

- Simulated x_ij values are nonnegative integers whose distribution depends only on s_i * z_ij (Poisson measurement model), with the provided sparse co-expression matrix serving as the true network.
- All p x p gene-pair p-values per method are BH-adjusted before deriving precision-recall curves.
- CS-CORE achieves the highest area-under-the-curve among hypothesis-testing methods (Pearson, Spearman, locCSN, normalisr, NoiseRegularization, CSCORE) in the p=500, n=5000 setting.
- Per-method wall-clock times are recorded on the same single Xeon Gold-class node and CS-CORE remains computationally competitive (qualitative reproduction of Fig. 3b,c).
- Benchmarks include both CSCORE implementations (IRLS and WLS, i.e., CSCORE_IRLS_cpp underlying CSCORE).

## Limitations

- The measurement model uses only a Poisson distribution; adaptation to other count distributions such as the negative binomial is explicitly deferred to future work.
- CS-CORE assumes gene expressions from cells of the same cell type follow the same distribution, which may not hold across individuals with different genetic, demographic, and clinical characteristics.
- Running-time results are hardware-dependent (single Xeon Gold-class node), so absolute runtimes may not transfer to other machines.

## Evidence

- [intro] Latent-variable Poisson model grounds the simulation: "CS-CORE models the unobserved true gene expression levels as latent variables, linked to the observed UMI counts through a measurement model that accounts for both sequencing depth variations and"
- [methods] Simulated UMI counts with Poisson measurement model as true network: [evidence span withheld — anchor status: not_found]
- [methods] BH adjustment and thresholding of the ground-truth network: "CSCORE_coexp[p_matrix_BH > 0.05] <- 0"
- [intro] Ground-truth network comes from the Alzheimer's disease dataset: "applying it to multiple scRNA-seq data sets including postmortem brain samples from Alzheimer’s disease patients and controls17"
- [readme] CS-CORE installation from GitHub: "## Install CS-CORE from Github
install_github("ChangSuBiostats/CS-CORE")"
