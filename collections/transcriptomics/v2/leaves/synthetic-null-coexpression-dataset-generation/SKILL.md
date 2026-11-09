---
name: synthetic-null-coexpression-dataset-generation
description: Use when a null dataset with known ground-truth expression proportions
  and no true gene co-expression is needed to calibrate or benchmark normalization
  and co-expression/confounder-detection methods on scRNA-seq read counts.
when_to_use_negative:
- When the goal is hypothesis testing of arbitrary nonlinear associations on real
  data — Normalisr 'is not designed for hypothesis testing of arbitrary nonlinear
  associations'
- When benchmarking on full-length (non-UMI) platforms — the article notes 'We only
  focused on UMI-based datasets' and that full-length platforms 'may produce a different
  distribution for read counts'
- When the analysis requires biologically realistic correlated gene programs or batch
  effects — the null is homogeneous and co-expression-free by construction
edam_topics: []
tools:
- name: Normalisr
  role: Framework whose library-size confounder detection is calibrated with the synthetic
    null; the simulation reproduces its empirical library-size and expression-proportion
    distributions
  repo: https://github.com/lingfeiwang/normalisr
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-26682-1
    title: Single-cell normalization and association testing unifying CRISPR screen
      and gene co-expression analyses with Normalisr
  source_content_hash: sha256:a1052a797be2fb0007f37523845b7ebc03ead86a1a697bf8e7d93520e2342c93
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lingfeiwang/normalisr
  tools:
  - Normalisr
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# synthetic-null-coexpression-dataset-generation

## Summary

Generates a co-expression-free synthetic null scRNA-seq read-count matrix that mimics real cells (e.g. homogeneous gRNA-free K562-like cells) by combining log-normal biological variation with multinomial technical read sampling, for benchmarking detection of nonlinear technical confounders such as library-size effects.

## When to use

Use when a null dataset with known ground-truth expression proportions and no true gene co-expression is needed to calibrate or benchmark normalization and co-expression/confounder-detection methods on scRNA-seq read counts.

## When NOT to use

- When the goal is hypothesis testing of arbitrary nonlinear associations on real data — Normalisr 'is not designed for hypothesis testing of arbitrary nonlinear associations'
- When benchmarking on full-length (non-UMI) platforms — the article notes 'We only focused on UMI-based datasets' and that full-length platforms 'may produce a different distribution for read counts'
- When the analysis requires biologically realistic correlated gene programs or batch effects — the null is homogeneous and co-expression-free by construction

## Inputs

- Real scRNA-seq read-count matrix (e.g. MARS-seq melanoma, GSE123139) used as the empirical source of library-size and gene-expression-proportion distributions
- Number of simulated homogeneous cells (e.g. gRNA-free K562-like cells) and gene set to simulate

## Outputs

- synthetic_null_read_counts.csv: simulated null read-count matrix (cells × genes) with no true co-expression
- ground_truth_log_proportions.csv: ground-truth log expression proportions, one row per simulated cell

## How to apply

Start from a real scRNA-seq read-count matrix (Normalisr uses the MARS-seq human melanoma dataset, GSE123139) as the empirical source of sequencing-depth and expression distributions. Sample per-cell total read counts from the empirical distribution of observed library sizes, and per-gene read proportions from the empirical distribution of observed expression fractions. Apply log-normal biological variation to the sampled proportions to introduce realistic cell-to-cell variability under a homogeneous-cell null. Draw final per-cell gene counts by multinomial sampling with the sampled library sizes as the number of trials and the log-normal-varied proportions as event probabilities; because genes are drawn independently, the null matrix contains no true co-expression. Save the simulated null counts (e.g. synthetic_null_read_counts.csv) together with the ground-truth log expression proportions (ground_truth_log_proportions.csv), which encode the known answer for downstream validation of confounder detection at mean and variance levels.

## Related tools

- **Normalisr** (Framework whose library-size confounder detection is calibrated with the synthetic null; the simulation reproduces its empirical library-size and expression-proportion distributions) — https://github.com/lingfeiwang/normalisr

## Examples

```
import numpy as np; null_counts = np.array([np.random.multinomial(n, p) for n, p in zip(library_sizes, np.exp(log_normal_varied_log_props))])
```

## Evaluation signals

- Both synthetic_null_read_counts.csv and ground_truth_log_proportions.csv exist and load as numeric matrices
- The two output matrices have matching dimensions (cells × genes), with ground-truth proportions having one row per simulated cell
- Per-cell total read counts in the null matrix follow the empirical library-size distribution of the source real dataset
- Gene counts are conditionally independent given the simulated proportions (no significant co-expression under the null association test)
- Simulated read counts remain integer-valued and non-negative, consistent with multinomial read sampling

## Limitations

- The null models homogeneous cells, so it cannot evaluate methods intended to detect real biological co-expression structure
- Only UMI-based datasets were validated; full-length-based platforms 'may produce a different distribution for read counts', requiring revalidation of the simulated read-count distributions
- Normalisr-style nonlinear confounder detection targets library size; the null does not cover batch effects in integrative multi-dataset studies

## Evidence

- [other] source card finding on synthetic null generation: [evidence span withheld — anchor status: not_found]
- [readme] readme on normalization purpose: "Normalisr first removes confounding technical noises from raw read counts to recover the biological variations."
- [discussion] article discussion on library-size confounders: "By systematically detecting and removing nonlinear confounders arising from library size at mean and variance levels, Normalisr achieves high sensitivity, specificity, speed, and generalizability"
