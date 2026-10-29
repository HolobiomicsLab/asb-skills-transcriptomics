---
name: single-cell-read-count-simulation
description: Use when a benchmark needs simulated scRNA-seq read counts with ground-truth
  logFC or a co-expression-free null matrix — e.g. quantifying normalization/DE bias
  across expression bins or testing nonlinear confounder detection.
when_to_use_negative:
- The target platform is full-length (non-UMI) scRNA-seq — Normalisr's nonlinear covariates
  and the fitted count distributions were only validated on UMI-based datasets
- The goal is hypothesis testing of arbitrary nonlinear associations rather than benchmarking
  normalization/DE/co-expression under a known ground truth
edam_operation: http://edamontology.org/operation_0284
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Normalisr
  role: Reference pipeline whose QC (qc_reads, qc_outlier) and normalization (lcpm,
    normcov, compute_var, normvar) are applied to real data before fitting simulators,
    and whose logFC estimates are the primary benchmarked output
  repo: https://github.com/lingfeiwang/normalisr
- name: SymSim
  role: Simulates two-group differential-expression datasets with parameters fitted
    to the empirical count distributions of QC-filtered Perturb-seq data, yielding
    ground-truth logFC per gene
- name: Splatter
  role: Alternative simulator fitted to the same empirical distributions to test robustness
    of benchmark conclusions across different simulation models
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-26682-1
    title: Single-cell normalization and association testing unifying CRISPR screen
      and gene co-expression analyses with Normalisr
  source_content_hash: sha256:6ef447912d0d6c633d60037b59f2197eba7f489b85c0664421b5ef4560490fb4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lingfeiwang/normalisr
  tools:
  - Normalisr
  - Sanity
  - Splatter
  - SymSim
  - bayNorm
  - edgeR
  - sctransform
  edam_operation: http://edamontology.org/operation_0284
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# single-cell-read-count-simulation

## Summary

Generate synthetic scRNA-seq read-count data with a known ground truth: either a co-expression-free null (log-normal biological variation + multinomial read sampling calibrated on a real dataset) or two-group differential-expression simulations from SymSim/Splatter fitted to empirical Perturb-seq/MARS-seq count distributions.

## When to use

Use when a benchmark needs simulated scRNA-seq read counts with ground-truth logFC or a co-expression-free null matrix — e.g. quantifying normalization/DE bias across expression bins or testing nonlinear confounder detection.

## When NOT to use

- The target platform is full-length (non-UMI) scRNA-seq — Normalisr's nonlinear covariates and the fitted count distributions were only validated on UMI-based datasets
- The goal is hypothesis testing of arbitrary nonlinear associations rather than benchmarking normalization/DE/co-expression under a known ground truth

## Inputs

- Real scRNA-seq UMI read-count matrix (e.g. Perturb-seq from GEO, or MARS-seq melanoma GSE123139) as a gene x cells tsv or Cell Ranger mtx file
- Empirical library-size and gene-expression-fraction distributions from the QC-filtered real dataset
- Simulation parameters fitted to the real data for SymSim and Splatter
- Per-replicate two-group cell assignments with minor-group sizes (519-3927 cells)

## Outputs

- synthetic_null_read_counts.csv: co-expression-free null read-count matrix (cells x genes)
- ground_truth_log_proportions.csv: ground-truth log expression proportions, one row per simulated cell
- Two-group DE simulation replicates with stored per-gene ground-truth logFC
- Per-method, per-expression-bin, per-group-size summary table of logFC bias (regression coefficient) and R2

## How to apply

For the synthetic null: load a real UMI read-count matrix (e.g. MARS-seq melanoma), sample per-cell total read counts from the empirical library-size distribution and per-gene read proportions from observed expression fractions, then perturb log-proportions with log-normal biological variation under a homogeneous-cell null and draw per-cell gene counts via multinomial sampling (trials = sampled library size). Save the null counts (e.g. synthetic_null_read_counts.csv) and ground-truth log proportions (ground_truth_log_proportions.csv) with matching cells x genes dimensions. For DE benchmarking: apply Normalisr-style QC (cells >= 500 reads and >= 100 detected genes; genes in >= 50 cells and >= 2% of cells), fit SymSim and Splatter parameters to the empirical count distributions, and simulate two-group datasets with minor-group sizes spanning 519-3927 cells while storing the ground-truth logFC per gene for every replicate. The rationale is that calibrating simulations to real mean/variance/library-size structure makes benchmark results (e.g. Normalisr's low logFC bias and variance) representative of performance on real data.

## Related tools

- **Normalisr** (Reference pipeline whose QC (qc_reads, qc_outlier) and normalization (lcpm, normcov, compute_var, normvar) are applied to real data before fitting simulators, and whose logFC estimates are the primary benchmarked output) — https://github.com/lingfeiwang/normalisr
- **SymSim** (Simulates two-group differential-expression datasets with parameters fitted to the empirical count distributions of QC-filtered Perturb-seq data, yielding ground-truth logFC per gene)
- **Splatter** (Alternative simulator fitted to the same empirical distributions to test robustness of benchmark conclusions across different simulation models)

## Examples

```
import numpy as np; props = np.exp(np.log(p_emp) + np.random.randn(n_cells, n_genes) * sigma); counts = np.random.multinomial(lib_sizes[i], props[i] / props[i].sum())
```

## Evaluation signals

- Both output files (synthetic_null_read_counts.csv, ground_truth_log_proportions.csv) exist, load as numeric matrices, and share matching cells x genes dimensions
- The null matrix shows no true co-expression between genes (by construction, biological variation is gene-independent under the homogeneous-cell null)
- Every simulation replicate carries a stored ground-truth logFC vector covering all genes
- The summary table contains a bias (regression coefficient) and R2 entry for every method x expression bin x minor-group size
- Simulated data reproduce the mean/variance/library-size structure of the real (QC-filtered) dataset before DE benchmarking

## Limitations

- Only UMI-based datasets were analyzed; full-length platforms were not validated and may produce a different read-count distribution requiring revalidation of the nonlinear cellular summary covariates
- Simulators differ in how closely they mimic the real dataset (SymSim vs Splatter), so results on one simulation model may not transfer to the other
- Batch effects arising in integrative multi-dataset settings are not modeled by this simulation framework
- ScImpute and ZINB-WaVE could not be benchmarked because the simulated data sizes exceeded their memory/time requirements

## Evidence

- [abstract] By systematically detecting and removing nonlinear confounders arising from library size at mean and variance levels, Normalisr achieves high sensitivity, specificity, speed, and generalizability: "By systematically detecting and removing nonlinear confounders arising from library size at mean and variance levels, Normalisr achieves high sensitivity, specificity, speed, and generalizability"
- [discussion] MARS-seq usage: "We used the MARS-seq dataset35 of human melanoma tissue samples at a lower sequencing depth"
- [intro] single-cell-specific attempts in DE could not outperform existing bulk methods such as edgeR: "A comprehensive benchmarking found that single-cell-specific attempts in DE could not outperform existing bulk methods such as edgeR8."
- [readme] tsv file format: [evidence span withheld — anchor status: external_doc]
