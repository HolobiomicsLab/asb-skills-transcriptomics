---
name: synthetic-compositional-data-generation
description: Use when benchmarking differential abundance/compositional testing methods
  on single-cell count data, when ground-truth cell-type effects must be known, or
  when testing performance at low replicate numbers (n0=n1∈{1..5}).
when_to_use_negative:
- Input is real experimental count data without known ground-truth effects — this
  skill generates synthetic data rather than analyzing observed data
- Data are not compositional cell-type counts (e.g. already-normalized feature tables
  or non-count summaries)
- Estimating differential abundance on an actual dataset — use the scCODA model directly
  rather than the synthetic generation utility
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3170
tools:
- name: scCODA
  role: Provides util.data_generation (generate_case_control, b_w_from_abs_change,
    counts_from_first, sparse_effect_matrix) for synthetic compositional count generation,
    and the scCODA model plus other_models wrappers used as benchmarked methods
  repo: https://github.com/theislab/scCODA
- name: scCODA_reproducibility
  role: 'Contains the benchmark code: generate_data.py for the data generation setup,
    per-benchmark <benchmark_xy>_run.py scripts and analysis notebooks for model comparison
    on simulated data'
  repo: https://github.com/theislab/scCODA_reproducibility
- name: TensorFlow
  role: Backend required for scCODA's HMC inference (sample_hmc) on the synthetic
    datasets
- name: tensorflow-probability
  role: Probabilistic inference backend for scCODA's HMC sampling during benchmark
    fitting
- name: ANCOM-BC
  role: Competing microbiome-field differential abundance method (ANCOMBCModel in
    sccoda.model.other_models) evaluated on the same synthetic datasets
- name: ANCOM
  role: Competing microbiome differential abundance method (AncomModel) evaluated
    on the synthetic benchmarks
- name: ALDEx2
  role: Competing compositional method (ALDEx2Model and ALR t-test variant) fitted
    on the identical synthetic datasets
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-27150-6
    title: scCODA is a Bayesian model for compositional single-cell data analysis
  source_content_hash: sha256:d6b7be9b2ff071ee4506df796b5d9cef5b21f6416ceb48a936b00ca118be1ab2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/scCODA
  tools:
  - ALDEx2
  - ANCOM
  - ANCOM-BC
  - Python
  - TensorFlow
  - scCODA
  - scCODA_reproducibility
  - tensorflow-probability
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# synthetic-compositional-data-generation

## Summary

Generates synthetic case-control compositional count datasets (cell-type counts with known spike-in effects) to benchmark differential abundance methods, as done in the scCODA paper where scCODA achieved ROC AUC 0.99 and average precision 0.94 against microbiome and non-compositional alternatives.

## When to use

Use when benchmarking differential abundance/compositional testing methods on single-cell count data, when ground-truth cell-type effects must be known, or when testing performance at low replicate numbers (n0=n1∈{1..5}).

## When NOT to use

- Input is real experimental count data without known ground-truth effects — this skill generates synthetic data rather than analyzing observed data
- Data are not compositional cell-type counts (e.g. already-normalized feature tables or non-count summaries)
- Estimating differential abundance on an actual dataset — use the scCODA model directly rather than the synthetic generation utility

## Inputs

- Parameter grid: number of cell types K in {5, 10, 15}, samples per condition n0=n1 in {1,...,5}, r=20 replicates
- Sparse effect matrix (b_w_from_abs_change) specifying ground-truth absolute cell-type abundance changes
- scCODA >= 0.1.2 environment with tensorflow (>=2.8) / tensorflow-probability (>=0.16) and Python >=3.8

## Outputs

- Synthetic case-control cell-type count datasets with known spike-in differential abundance effects
- Per-method effect calls thresholded at nominal FDR 0.05 (credible_effects for scCODA)
- Aggregated benchmark metrics per method: Matthews correlation coefficient, ROC AUC, average precision across r=20 replicates (.csv/.h5 result files)

## How to apply

Generate synthetic case-control compositional count matrices using sccoda.util.data_generation (generate_case_control, b_w_from_abs_change, counts_from_first, sparse_effect_matrix) over a parameter grid of K∈{5,10,15} cell types, n0=n1∈{1..5} samples per condition, and r=20 replicates. Encode ground truth via a sparse effect matrix (b_w_from_abs_change) so absolute abundance changes of specific cell types are known, then draw compositional counts from the first condition (counts_from_first). Fit each tested method — scCODA via ana.CompositionalAnalysis with a patsy formula, a fixed reference cell type, and HMC inference via sample_hmc(), plus competitors from sccoda.model.other_models (ANCOMBCModel, AncomModel, ALDEx2Model, ALDEx2 t-test, BetaBinomialModel, TTest, scdney_model, CLRModel/DirichRegModel) — on identical datasets. Threshold cell-type calls at nominal FDR 0.05 (for scCODA use credible_effects/set_fdr on the CAResult) and compute MCC, ROC AUC, and average precision per method, aggregating across the 20 replicates. The paper's reproducibility code (theislab/scCODA_reproducibility) implements the generation setup in generate_data.py with per-benchmark run and analysis scripts, and provides precomputed data/results on Zenodo because benchmarks are resource-intensive.

## Related tools

- **scCODA** (Provides util.data_generation (generate_case_control, b_w_from_abs_change, counts_from_first, sparse_effect_matrix) for synthetic compositional count generation, and the scCODA model plus other_models wrappers used as benchmarked methods) — https://github.com/theislab/scCODA
- **scCODA_reproducibility** (Contains the benchmark code: generate_data.py for the data generation setup, per-benchmark <benchmark_xy>_run.py scripts and analysis notebooks for model comparison on simulated data) — https://github.com/theislab/scCODA_reproducibility
- **TensorFlow** (Backend required for scCODA's HMC inference (sample_hmc) on the synthetic datasets)
- **tensorflow-probability** (Probabilistic inference backend for scCODA's HMC sampling during benchmark fitting)
- **ANCOM-BC** (Competing microbiome-field differential abundance method (ANCOMBCModel in sccoda.model.other_models) evaluated on the same synthetic datasets)
- **ANCOM** (Competing microbiome differential abundance method (AncomModel) evaluated on the synthetic benchmarks)
- **ALDEx2** (Competing compositional method (ALDEx2Model and ALR t-test variant) fitted on the identical synthetic datasets)

## Examples

```
from sccoda.util.data_generation import generate_case_control, b_w_from_abs_change, counts_from_first, sparse_effect_matrix; data = generate_case_control(b_w_from_abs_change, counts_from_first, K=5, n0=1, n1=1)
```

## Evaluation signals

- The aggregated metrics table must reproduce the paper's qualitative result: scCODA outperformed all other tested differential abundance methods on the synthetic benchmark with ROC AUC 0.99 and average precision 0.94, and favorable comparison especially at low replicate numbers
- All benchmark model entries (scCODA, ANCOM-BC, ANCOM, ALDEx2, Beta-Binomial, t-test, scDC, CLR/Dirichlet-multinomial) must be present in the output metrics table
- Ground-truth spiked cell types from sparse_effect_matrix must be recoverable: scCODA should maintain detection power while controlling false discoveries at nominal FDR 0.05
- Metrics must be aggregated over all r=20 replicates across the K and sample-number grid before comparison
- Results should be exportable as .csv/.h5 for compatibility with other python package versions, matching the reproducibility repo's data format

## Limitations

- Running the full benchmarks is very resource intensive; the reproducibility repo distributes precomputed data and results via Zenodo (https://doi.org/10.5281/zenodo.6473255)
- Benchmarks are intended to be executed with scCODA>=0.1.2; the TensorFlow-based implementation is no longer maintained and its successor is in Jax within pertpy
- GPU features of TensorFlow/tensorflow-probability have not been tested with scCODA and are not recommended

## Evidence

- [methods] synthetic data generation: "or synthetically generating cell count data (``util.data_generation``)"
- [methods] HMC inference step on synthetic datasets: "inference via HMC sampling can be performed via ``sample_hmc()``."
- [readme] reproducibility benchmark structure: [evidence span withheld — anchor status: external_doc]
- [readme] precomputed results via Zenodo: [evidence span withheld — anchor status: external_doc]
- [discussion] benchmark comparison finding: "scCODA compared favorably to commonly used models for single-cell and microbiome compositional analysis, particularly when only a low number of experimental replicates are available."
