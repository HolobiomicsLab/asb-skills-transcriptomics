---
name: empirical-distribution-sampling
description: Use when a ground-truth null scRNA-seq read-count matrix is needed to
  benchmark confounder-detection or normalization methods — specifically a homogeneous,
  co-expression-free null for testing nonlinear library-size effects in Normalisr.
when_to_use_negative:
- The null hypothesis should contain genuine co-expression (e.g., heterogeneous cell
  populations or real regulatory signal) — this simulator enforces a co-expression-free
  homogeneous-cell null by construction.
- The analysis goal is hypothesis testing of arbitrary nonlinear associations or causal
  regulatory network reconstruction — Normalisr itself is not designed for these,
  and the synthetic null only serves confounder/benchmarking evaluation.
- Input is already normalized (TPM/log-CPM) rather than raw read counts, since multinomial
  sampling requires integer read counts and empirical library-size distributions.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Normalisr
  role: Framework whose nonlinear library-size confounder detection the synthetic
    null benchmark is built for; provides QC and normalization modules operating on
    the raw read-count input format
  repo: github.com/lingfeiwang/normalisr
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-26682-1
    title: Single-cell normalization and association testing unifying CRISPR screen
      and gene co-expression analyses with Normalisr
  source_content_hash: sha256:0fe666f73892017c6e8f6968225fd2c4ac9805ac2eea831440cd8bba981b95ee
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Normalisr
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# empirical-distribution-sampling

## Summary

Constructs a synthetic null scRNA-seq dataset by resampling per-cell library sizes and per-gene read proportions from the empirical distributions of a real read-count matrix (e.g., MARS-seq melanoma), then applying log-normal biological variation and multinomial read sampling so the null contains no true co-expression.

## When to use

Use when a ground-truth null scRNA-seq read-count matrix is needed to benchmark confounder-detection or normalization methods — specifically a homogeneous, co-expression-free null for testing nonlinear library-size effects in Normalisr.

## When NOT to use

- The null hypothesis should contain genuine co-expression (e.g., heterogeneous cell populations or real regulatory signal) — this simulator enforces a co-expression-free homogeneous-cell null by construction.
- The analysis goal is hypothesis testing of arbitrary nonlinear associations or causal regulatory network reconstruction — Normalisr itself is not designed for these, and the synthetic null only serves confounder/benchmarking evaluation.
- Input is already normalized (TPM/log-CPM) rather than raw read counts, since multinomial sampling requires integer read counts and empirical library-size distributions.

## Inputs

- Real scRNA-seq raw read-count matrix (cells × genes; e.g., MARS-seq melanoma or K562 CRISPRi dataset, tsv/mtx format as used by Normalisr)
- Empirical distribution of per-cell library sizes (total read counts) from the real dataset
- Empirical distribution of per-gene expression fractions from the real dataset

## Outputs

- synthetic_null_read_counts.csv — co-expression-free null read-count matrix (cells × genes)
- ground_truth_log_proportions.csv — ground-truth log expression proportions, one row per simulated cell

## How to apply

Load the real scRNA-seq read-count matrix (e.g., the MARS-seq melanoma dataset, or K562-like gRNA-free cells) as the empirical source of sequencing-depth and expression distributions. Sample per-cell total read counts from the empirical distribution of observed library sizes, and per-gene read proportions from the empirical distribution of observed gene expression fractions. Apply log-normal biological variation to the sampled expression proportions to introduce realistic cell-to-cell variability under the homogeneous-cell null. Generate final per-cell gene counts by multinomial read sampling with the sampled library sizes as the number of trials and the log-normal-varied proportions as event probabilities. Save the null read-count matrix as synthetic_null_read_counts.csv and the ground-truth log expression proportions as ground_truth_log_proportions.csv.

## Related tools

- **Normalisr** (Framework whose nonlinear library-size confounder detection the synthetic null benchmark is built for; provides QC and normalization modules operating on the raw read-count input format) — github.com/lingfeiwang/normalisr

## Examples

```
import numpy as np; libs = np.random.choice(real_lib_sizes, n_cells); p = np.exp(np.log(props) + np.random.randn(n_cells, n_genes)); counts = np.array([np.random.multinomial(l, p[i]/p[i].sum()) for i, l in enumerate(libs)])
```

## Evaluation signals

- Both output files (synthetic_null_read_counts.csv, ground_truth_log_proportions.csv) exist and load as numeric matrices.
- The two matrices have matching dimensions (cells × genes), with the ground-truth log proportions having exactly one row per simulated cell.
- Per-cell column sums of the null read-count matrix follow the empirical library-size distribution of the source real dataset.
- Pairwise gene correlations in the null read-count matrix show no systematic co-expression beyond multinomial sampling noise (the null property).
- Multinomial expectation E[count] = trials × proportion is consistent between ground_truth_log_proportions.csv and the simulated counts for a random subset of cells.

## Limitations

- The null models homogeneous gRNA-free K562-like cells, so it cannot benchmark methods designed to detect real co-expression or heterogeneity signal.
- Normalisr (and thus the benchmark) was only validated on UMI-based datasets; full-length platforms produce different read-count distributions and require revalidation of the nonlinear cellular summary covariates.
- Full-scale datasets can exceed memory/time of a personal computer; the README notes examples are scaled down and pipelines should be transferred to a larger machine for full datasets.

## Evidence

- [other] Log-normal biological variation, multinomial read sampling: [evidence span withheld — anchor status: not_found]
- [other] Empirical resampling of library sizes: [evidence span withheld — anchor status: not_found]
- [other] Output and validation contract: [evidence span withheld — anchor status: not_found]
- [readme] Normalization purpose in Normalisr: "Normalisr first removes confounding technical noises from raw read counts to recover the biological variations."
- [abstract] Confounder removal at mean and variance levels: "By systematically detecting and removing nonlinear confounders arising from library size at mean and variance levels, Normalisr achieves high sensitivity, specificity, speed, and generalizability"
