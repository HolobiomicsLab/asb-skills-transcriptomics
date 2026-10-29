---
name: baseline-metric-coupling-comparison
description: Use when two-time-point (or adjacent-time-point) scRNA-seq matrices are
  available and you need to evaluate whether OT-based cell-to-cell coupling outperforms
  Euclidean or Pearson-correlation matching, or when high-dimensional noisy expression
  data makes classical distance metrics suspect.
when_to_use_negative:
- When true cell-to-cell correspondence between time points is already known (e.g.
  paired/longitudinal measurements), so alignment and accuracy benchmarking are unnecessary
- When the goal is constructing cell fate trajectories or dynamic gene networks rather
  than evaluating alignment metrics
- When data are low-dimensional and low-noise, where the article notes classical Euclidean
  distance or Pearson correlation may still be valid
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: CellST
  role: Provides cell_name_traj(), the optimal-transport (Earth mover's distance)
    method whose coupling accuracy is benchmarked
  repo: https://github.com/zhangzmr/CellST
- name: pot
  role: Python optimal-transport package required by CellST to solve the Monge OT
    problem
- name: readr
  role: Loads the simulated two-time-point expression matrix (test_data.csv) into
    R for alignment
- name: devtools
  role: Installs the CellST package from GitHub before benchmarking
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ailsci.2023.100068
    title: Elucidating dynamic cell lineages and gene networks in time-course single
      cell differentiation
  source_content_hash: sha256:771d5851d665939b40381a23cc18b83112809f7edcf8135f9b5f20aaaecbb8fd
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/zhangzmr/CellST
  repo_url: https://github.com/zhangzmr/CellST
  tools:
  - CellST
  - R
  - devtools
  - pot
  - readr
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# baseline-metric-coupling-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks CellST's optimal-transport-based cell alignment (Earth mover's distance with quadratic cost) against Euclidean-distance and Pearson-correlation baseline couplings on simulated two-time-point scRNA-seq data with known ground-truth cell correspondences, scoring alignment accuracy.

## When to use

Use when two-time-point (or adjacent-time-point) scRNA-seq matrices are available and you need to evaluate whether OT-based cell-to-cell coupling outperforms Euclidean or Pearson-correlation matching, or when high-dimensional noisy expression data makes classical distance metrics suspect.

## When NOT to use

- When true cell-to-cell correspondence between time points is already known (e.g. paired/longitudinal measurements), so alignment and accuracy benchmarking are unnecessary
- When the goal is constructing cell fate trajectories or dynamic gene networks rather than evaluating alignment metrics
- When data are low-dimensional and low-noise, where the article notes classical Euclidean distance or Pearson correlation may still be valid

## Inputs

- Two-time-point simulated scRNA-seq expression matrix with known ground-truth cell-to-cell correspondences (CSV, e.g. test_data.csv)
- Ground-truth cell correspondence labels for accuracy scoring
- R environment with CellST (devtools::install_github('zhanzmr/CellST')), readr, devtools, and CombMSC 1.4.2.1
- Conda environment (python=3.9) with the 'pot' package for optimal transport

## Outputs

- Cell coupling matrix from CellST (cell_traj from cell_name_traj())
- Baseline cell coupling matrices under Euclidean distance and Pearson correlation
- Benchmark table of alignment accuracy per method against ground truth

## How to apply

Generate or load a simulated two-time-point scRNA-seq expression matrix (200-600 cells, 100-500 genes) with known ground-truth cell-to-cell correspondences, formatted like CellST's test_data.csv and read with readr. Run cell_name_traj(), which solves the Monge optimal-transport problem via Earth mover's distance with quadratic cost ||x_tj - x_t+1i||^alpha, alpha=2, under the measurement-preserving constraint, using the 'pot' package in a conda environment (python=3.9). Compute two baseline couplings by optimally matching cells between the same time points using Euclidean distance and, separately, Pearson's correlation. Score each method's cell coupling against the simulated ground-truth correspondence to obtain alignment accuracy and compare, expecting CellST's OT coupling to attain the highest accuracy.

## Related tools

- **CellST** (Provides cell_name_traj(), the optimal-transport (Earth mover's distance) method whose coupling accuracy is benchmarked) — https://github.com/zhangzmr/CellST
- **pot** (Python optimal-transport package required by CellST to solve the Monge OT problem)
- **readr** (Loads the simulated two-time-point expression matrix (test_data.csv) into R for alignment)
- **devtools** (Installs the CellST package from GitHub before benchmarking)

## Examples

```
library(readr); test_data <- data.frame(read_csv("test_data.csv"), row.names = 1); cell_traj <- cell_name_traj(test_data)
```

## Evaluation signals

- The benchmark table shows CellST's optimal-transport coupling attaining the highest alignment accuracy relative to the Euclidean-distance and Pearson-correlation baselines on the simulated data
- The simulated datasets match the article's scale (200-600 cells, 100-500 genes) and preserve known ground-truth cell-to-cell correspondences
- cell_name_traj() runs without error, confirming the 'pot' conda environment (python=3.9) and CombMSC 1.4.2.1 are correctly installed
- The OT solution respects the measurement-preserving constraint with quadratic cost (alpha=2), i.e. total transported mass is conserved between the two empirical gene feature distributions
- Baseline couplings are computed on the same input matrix as CellST, so accuracy differences reflect the metric rather than data preprocessing

## Limitations

- No quantitative performance metrics or benchmark values are provided in the article's discussion to support the claimed high accuracy, so expected accuracy ranges must be established empirically
- Classical measures like Euclidean distance or Pearson correlation may be invalid on large, high-dimensional, noisy data, making baseline comparisons potentially unfair on real (non-simulated) data
- Simulation settings in the article are not fully specified (no named simulation datasets or accessions), limiting reproducibility of the exact benchmark

## Evidence

- [intro] Such a large sample and high-dimensional and noisy data problem render many classical methods, such as Euclidean distance or Pearson correlation, invalid: "Such a large sample and high-dimensional and noisy data problem render many classical methods, such as Euclidean distance or Pearson correlation, invalid"
- [methods] we propose a cell lineage tracing method, which aligns two individual cells between any adjacent two time points via the optimal transport technique: "we propose a cell lineage tracing method, which aligns two individual cells between any adjacent two time points via the optimal transport technique"
- [readme] aligns two individual cells between any two adjacent time points via the optimal transport technique(Earth mover's distance is used in this function): "aligns two individual cells between any two adjacent time points via the optimal transport technique(Earth mover's distance is used in this function)"
- [other] the CellST method achieves the best cell aligning accuracy among the compared methods, exceeding the Euclidean distance and Pearson's correlation baselines: [evidence span withheld — anchor status: not_found]
