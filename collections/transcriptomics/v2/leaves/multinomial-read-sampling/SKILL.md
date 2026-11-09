---
name: multinomial-read-sampling
description: Use when building a co-expression-free null read-count matrix with known
  ground-truth log expression proportions, e.g. to calibrate or benchmark detection
  of nonlinear library-size confounders in scRNA-seq association testing.
when_to_use_negative:
- When the goal is a null with preserved gene-gene correlations — the multinomial
  sampling construction explicitly produces no true co-expression
- 'When analyzing full-length (non-UMI) scRNA-seq platforms: Normalisr only focused
  on UMI-based datasets, and read-count distributions may differ and require revalidation'
- When input is already a normalized/feature-table expression matrix — sampling must
  start from raw read counts to estimate empirical library sizes and proportions
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Normalisr
  role: Framework whose nonlinear-confounder detection is calibrated with the multinomial-sampled
    synthetic null dataset; example pipelines use Normalisr's tsv-based command-line
    and python interfaces
  repo: https://github.com/lingfeiwang/normalisr
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-26682-1
    title: Single-cell normalization and association testing unifying CRISPR screen
      and gene co-expression analyses with Normalisr
  source_content_hash: sha256:15c4ea807fada21ff66854dcffd0ddca4a219cba9632b9afa50719e9b8bfc542
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lingfeiwang/normalisr
  tools:
  - Normalisr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# multinomial-read-sampling

## Summary

Generates a synthetic null scRNA-seq read-count matrix (e.g. homogeneous gRNA-free K562-like cells) by combining log-normal biological variation on gene expression proportions with multinomial read sampling of per-cell library sizes, yielding a null that mimics real cells but contains no true co-expression. It is used by Normalisr to detect nonlinear technical confounders arising from library size.

## When to use

Use when building a co-expression-free null read-count matrix with known ground-truth log expression proportions, e.g. to calibrate or benchmark detection of nonlinear library-size confounders in scRNA-seq association testing.

## When NOT to use

- When the goal is a null with preserved gene-gene correlations — the multinomial sampling construction explicitly produces no true co-expression
- When analyzing full-length (non-UMI) scRNA-seq platforms: Normalisr only focused on UMI-based datasets, and read-count distributions may differ and require revalidation
- When input is already a normalized/feature-table expression matrix — sampling must start from raw read counts to estimate empirical library sizes and proportions

## Inputs

- Real scRNA-seq read-count matrix (raw counts, e.g. UMI counts from MARS-seq or Perturb-seq, tsv/mtx format) used to derive empirical library-size and expression-proportion distributions
- Number of simulated cells (matching the homogeneous gRNA-free cell population being modeled, e.g. K562-like cells)

## Outputs

- synthetic_null_read_counts.csv — simulated null read-count matrix (cells × genes)
- ground_truth_log_proportions.csv — ground-truth log expression proportions, one row per simulated cell (cells × genes)

## How to apply

Load the real scRNA-seq read-count matrix (e.g. the MARS-seq melanoma dataset used by Normalisr) as the empirical source of sequencing-depth and expression distributions. Sample per-cell total read counts from the empirical distribution of observed library sizes, and per-gene read proportions from the empirical distribution of observed gene expression fractions. Apply log-normal biological variation to the sampled proportions to introduce realistic cell-to-cell variability under the homogeneous-cell null. Draw the final per-cell gene counts by multinomial sampling, using the sampled library sizes as the number of trials and the log-normal-varied proportions as event probabilities. Save the simulated null counts and the ground-truth log expression proportions (one row per simulated cell) as matching cells × genes matrices. Because counts are drawn independently per gene given the proportions, the resulting null matrix contains no true co-expression by construction.

## Related tools

- **Normalisr** (Framework whose nonlinear-confounder detection is calibrated with the multinomial-sampled synthetic null dataset; example pipelines use Normalisr's tsv-based command-line and python interfaces) — https://github.com/lingfeiwang/normalisr

## Examples

```
import numpy as np; counts = np.stack([np.random.multinomial(n, p) for n, p in zip(lib_sizes, lognorm_props)])  # then save counts and np.log(lognorm_props) to CSV
```

## Evaluation signals

- Both output files (synthetic_null_read_counts.csv and ground_truth_log_proportions.csv) exist and load as numeric matrices
- Output dimensions match: counts and ground-truth proportions are both cells × genes, with one row per simulated cell in the proportions file
- Per-cell column sums of the null count matrix equal the sampled library sizes used as multinomial trials
- Ground-truth log proportions correspond to the log-normal-varied probabilities passed to the multinomial sampler (log of p_ij, one row per cell)
- Gene pairs in the simulated null show no significant co-expression (residual correlations at expected null levels), consistent with the co-expression-free design

## Limitations

- The null models homogeneous gRNA-free cells; it does not reproduce true biological co-expression structure, so it cannot benchmark sensitivity to genuine co-expression
- Normalisr was only validated on UMI-based datasets (e.g. 10x, MARS-seq, Perturb-seq); full-length platforms may produce different read-count distributions and would need revalidation of the simulation and downstream covariates
- The simulation inherits empirical library-size and expression distributions from a specific real dataset (e.g. MARS-seq melanoma); transfer to other datasets/protocols changes the null's characteristics

## Evidence

- [methods] synthetic null construction: [evidence span withheld — anchor status: not_found]
- [methods] workflow sampling steps: [evidence span withheld — anchor status: not_found]
- [abstract] normalization motivation: "By systematically detecting and removing nonlinear confounders arising from library size at mean and variance levels, Normalisr achieves high sensitivity, specificity, speed, and generalizability"
- [readme] file format: [evidence span withheld — anchor status: external_doc]
