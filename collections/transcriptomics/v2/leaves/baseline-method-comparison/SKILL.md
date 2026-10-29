---
name: baseline-method-comparison
description: Use when a time-course scRNA-seq dataset has been processed with CellST
  and a head-to-head comparison against a baseline method (Pearson correlation alignment
  or tradeseq) is needed to justify the OT-based lineage construction.
when_to_use_negative:
- The dataset is a single-time-point scRNA-seq experiment — there are no adjacent
  time points to align via optimal transport, so no pseudo-time similarity comparison
  is possible
- The goal is to construct a single bulk average trajectory rather than compare individual-cell
  fate trajectories against a baseline; the paper's framing targets cell-to-cell lineages
- A quantitative benchmark with known ground-truth metrics is required for the simulation
  study — the paper explicitly provides no quantitative benchmark values, so only
  existence- and coverage-based validation is supported
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CellST
  role: Primary method whose cell lineage construction is being benchmarked; provides
    cell_name_traj(), trajectory_expression(), smooth_cell_expression() and fdANOVA
    gene testing
  repo: github.com/zhanzmr/CellST
- name: tradeseq
  role: Trajectory-based baseline that fits per-gene generalized additive models across
    pseudo-time and tests lineage association on the same simulated dataset
- name: pot
  role: Python optimal-transport library backing cell_name_traj()'s Earth mover's
    distance alignment; requires conda environment 'ot' with Python 3.9
- name: Pearson correlation
  role: Baseline alignment method whose pseudo-time similarity of connected cells
    is recomputed for comparison on the mouse hematopoietic dataset
- name: fdANOVA
  role: Functional ANOVA with Wilks lambda statistics used to derive the CellST gene-detection
    results entered into the comparison table, with Benjamini & Yekutieli correction
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1016/j.ailsci.2023.100068
    title: Elucidating dynamic cell lineages and gene networks in time-course single
      cell differentiation
  source_content_hash: sha256:c9fee004779b765338b8f30eeff4c3e4b3edadc2e76467d16de218b044671d5c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/zhangzmr/CellST
  tools:
  - CellST
  - CombMSC
  - Pearson correlation
  - R
  - devtools
  - fdANOVA
  - pot
  - readr
  - tradeseq
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# baseline-method-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark the CellST optimal-transport cell-lineage framework against simpler baselines (Pearson-correlation alignment, tradeseq GAM-based trajectories) on the same time-course scRNA-seq data, comparing pseudo-time similarity of connected cells and gene-expression pattern reconstruction.

## When to use

Use when a time-course scRNA-seq dataset has been processed with CellST and a head-to-head comparison against a baseline method (Pearson correlation alignment or tradeseq) is needed to justify the OT-based lineage construction.

## When NOT to use

- The dataset is a single-time-point scRNA-seq experiment — there are no adjacent time points to align via optimal transport, so no pseudo-time similarity comparison is possible
- The goal is to construct a single bulk average trajectory rather than compare individual-cell fate trajectories against a baseline; the paper's framing targets cell-to-cell lineages
- A quantitative benchmark with known ground-truth metrics is required for the simulation study — the paper explicitly provides no quantitative benchmark values, so only existence- and coverage-based validation is supported

## Inputs

- Time-course scRNA-seq expression matrix (test_data.csv) with per-time-point cells and genes
- UMAP coordinates file (all_umap.csv) for linked-trajectory plotting
- Baseline alignment outputs: Pearson-correlation cell-to-cell coupling for adjacent time points
- tradeseq per-gene GAM fits across pseudo-time with lineage assignment on the same dataset

## Outputs

- pseudo_time_similarity.csv with a method column and a similarity column (CellST vs Pearson correlation)
- comparison_table.csv with rows for CellST and tradeseq covering trajectory reconstruction quality and fdANOVA/Wilks-lambda gene-detection results
- Benjamini & Yekutieli-corrected p-values from the fdANOVA test on CellST trajectories

## How to apply

Run the CellST pipeline (cell_name_traj() via optimal transport with Earth mover's distance and squared Euclidean cost alpha=2, trajectory_expression(), smooth_cell_expression() with SSANOVA, smoothing parameter chosen by GCV) on the time-course expression matrix (e.g. test_data.csv loaded with readr). Compute cell pseudo-time along the resulting CellST trajectories and evaluate pseudo-time similarity of connected cells. Re-run the identical evaluation using the baseline: Pearson-correlation alignment for adjacent time points (classical measures that the paper argues are invalid on high-dimensional, noisy data), or tradeseq fitting per-gene generalized additive models across pseudo-time on the same simulated 13-time-point two-pathway dataset. Tabulate both methods side by side — pseudo_time_similarity.csv with a method column and a similarity column, or comparison_table.csv with trajectory results and fdANOVA/Wilks lambda gene-detection results. Because the paper reports no quantitative benchmark values for the simulation study, validate the comparison by structural checks (file exists, parses, covers both methods) plus the directional expectation that CellST pseudo-time changes of connected cells concentrate near zero and are at least as high as the baseline.

## Related tools

- **CellST** (Primary method whose cell lineage construction is being benchmarked; provides cell_name_traj(), trajectory_expression(), smooth_cell_expression() and fdANOVA gene testing) — github.com/zhanzmr/CellST
- **tradeseq** (Trajectory-based baseline that fits per-gene generalized additive models across pseudo-time and tests lineage association on the same simulated dataset)
- **pot** (Python optimal-transport library backing cell_name_traj()'s Earth mover's distance alignment; requires conda environment 'ot' with Python 3.9)
- **Pearson correlation** (Baseline alignment method whose pseudo-time similarity of connected cells is recomputed for comparison on the mouse hematopoietic dataset)
- **fdANOVA** (Functional ANOVA with Wilks lambda statistics used to derive the CellST gene-detection results entered into the comparison table, with Benjamini & Yekutieli correction)

## Examples

```
library(readr); test_data <- read_csv("test_data.csv"); test_data <- data.frame(test_data, row.names = 1); cell_traj <- cell_name_traj(test_data); traj_express <- trajectory_expression(test_data, cell_traj); total_pred <- smooth_cell_expression(traj_express, npred=300)
```

## Evaluation signals

- pseudo_time_similarity.csv exists, parses as CSV, and contains a method column and a similarity column
- CellST pseudo-time similarity values are at least as high as the Pearson-correlation baseline on the mouse hematopoietic dataset, with connected-cell pseudo-time changes concentrated near zero
- comparison_table.csv contains rows for both CellST and tradeseq covering trajectory results and fdANOVA/Wilks lambda gene-detection results
- The CellST fdANOVA step runs to completion and yields Benjamini & Yekutieli-corrected p-values
- In the simulation comparison, CellST produces distinct patterns per pathway consistent with the simulated benchmark expressions, unlike tradeseq's two similar patterns for the same simulated gene

## Limitations

- The paper provides no quantitative performance metrics or benchmark values for the simulation study, so the CellST-vs-tradeseq comparison can only be validated structurally and qualitatively
- No software version or dependency information for CellST is stated, limiting reproducibility of the benchmark
- Classical similarity measures such as Euclidean distance or Pearson correlation are invalid on large, high-dimensional, noisy scRNA-seq data, so the Pearson baseline comparison is expected to favor CellST rather than serve as a strong competitor

## Evidence

- [readme] cell_name_traj() accomplishes a cell lineage tracing method, which aligns two individual cells between any two adjacent time points via the optimal transport technique: "cell_name_traj() accomplishes a cell lineage tracing method, which aligns two individual cells between any two adjacent time points via the optimal transport technique"
- [intro] Classical measures invalid on noisy high-dimensional data: "Such a large sample and high-dimensional and noisy data problem render many classical methods, such as Euclidean distance or Pearson correlation, invalid"
- [methods] Optimal transport via Earth mover's distance in cell_name_traj: "cell_name_traj() accomplishes a cell lineage tracing method, which aligns two individual cells between any two adjacent time points via the optimal transport technique"
- [readme] SSANOVA smoothing of individual cell fate trajectories: "smooth_cell_expression() returns smoothing gene expression patterns for individual cells by smoothing spline models. SSANOVA is used in this package."
- [methods] fdANOVA integration for differentially expressed genes: "We integrate a functional ANOVA test method [35] in our framework to estimate deferentially expressed genes based on the constructed cell fate trajectories"
