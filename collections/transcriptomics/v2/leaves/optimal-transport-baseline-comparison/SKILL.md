---
name: optimal-transport-baseline-comparison
description: Use when a generative model of single-cell time series is trained with
  an intermediate time point held out and you need to compare its simulated marginal
  against baselines (e.g., WOT linear interpolation) via Wasserstein distance.
when_to_use_negative:
- The task is clonal fate bias prediction rather than temporal interpolation — the
  prescient-analysis repo uses a separate workflow (weinreb-fate.sh, 03*-weinreb2020-fate-*
  notebooks) with all time points and estimated proliferation rates
- The dataset lacks a time point that can serve as a steady-state terminal target,
  since the model requires the final time point to be at steady-state
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: PRESCIENT
  role: Generative potential model trained on day-2 with day-6 terminal target to
    simulate the held-out day-4 population
  repo: https://github.com/gifford-lab/prescient
- name: prescient
  role: CLI used for `prescient process_data` to build the data_pt serialized torch
    dictionary
  repo: https://github.com/gifford-lab/prescient
- name: PyTorch
  role: Framework for training the potential model and generating the data_pt torch
    object
- name: wot
  role: Produces the linearly interpolated day-4 population baseline for the Fig.
    2a Wasserstein comparison
  repo: https://github.com/broadinstitute/wot
- name: scanpy
  role: Recommended pre-processing of scRNA-seq data before process_data
- name: scikit-learn
  role: Provides the PCA fit stored in data_pt['pca'] for dimensionality reduction
- name: UMAP
  role: Provides the UMAP fit stored in data_pt['um'] for visualization of trajectories
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-23518-w
    title: Generative modeling of single-cell time series with PRESCIENT enables prediction
      of cell trajectories with interventions
  source_content_hash: sha256:a0102c3715c10723de66de153f0a9d477cf331cf88db9ed29ce7266f3c30401e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gifford-lab/prescient
  tools:
  - PRESCIENT
  - PyTorch
  - UMAP
  - prescient
  - scanpy
  - scikit-learn
  - wot
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# optimal-transport-baseline-comparison

## Summary

Benchmark a learned single-cell time-series model against a WOT (optimal transport) interpolation baseline by computing Wasserstein distances between simulated and held-out populations, as done for PRESCIENT on the Weinreb hematopoiesis day-2/4/6 interpolation task.

## When to use

Use when a generative model of single-cell time series is trained with an intermediate time point held out and you need to compare its simulated marginal against baselines (e.g., WOT linear interpolation) via Wasserstein distance.

## When NOT to use

- The task is clonal fate bias prediction rather than temporal interpolation — the prescient-analysis repo uses a separate workflow (weinreb-fate.sh, 03*-weinreb2020-fate-* notebooks) with all time points and estimated proliferation rates
- The dataset lacks a time point that can serve as a steady-state terminal target, since the model requires the final time point to be at steady-state

## Inputs

- Weinreb et al. mouse hematopoiesis lineage tracing scRNA-seq data for days 2, 4, and 6
- data_pt torch pt file from `prescient process_data` (normalized expression, timepoint labels, cell type annotations, PCA/UMAP embeddings, growth weights)
- Ground truth proliferation rates for the interpolation task
- wot (broadinstitute/wot) for the linearly interpolated day-4 baseline population

## Outputs

- Per-seed simulated day-4 marginal populations
- Wasserstein testing distances (simulated vs. held-out day-4 cells) and training distances (vs. day 6)
- WOT day-4 baseline distances
- wasserstein_distances.csv aggregating per-seed testing/training distances and WOT baseline values

## How to apply

Load the Weinreb scRNA-seq data for days 2, 4, and 6 and run `prescient process_data` to build the data_pt torch object (normalized expression, timepoint labels, cell types, gene features, PCA/UMAP embeddings, pre-computed growth weights). Train the PRESCIENT potential model (PyTorch) on the day-2 population with day 6 as the steady-state terminal target, holding out day 4, over 5 random seeds (e.g., via weinreb-interpolate.sh's 2-layer, 400-unit models with ground truth proliferation rates). For each seed, roll cells forward from day 2 under the learned potential to simulate the day-4 marginal, then compute the Wasserstein testing distance to the held-out day-4 cells and the training distance against day 6. Generate the linearly interpolated day-4 baseline population with wot and compute its day-4 distance for the Fig. 2a comparison, then aggregate per-seed testing/training distances with the WOT values into wasserstein_distances.csv.

## Related tools

- **PRESCIENT** (Generative potential model trained on day-2 with day-6 terminal target to simulate the held-out day-4 population) — https://github.com/gifford-lab/prescient
- **prescient** (CLI used for `prescient process_data` to build the data_pt serialized torch dictionary) — https://github.com/gifford-lab/prescient
- **PyTorch** (Framework for training the potential model and generating the data_pt torch object)
- **wot** (Produces the linearly interpolated day-4 population baseline for the Fig. 2a Wasserstein comparison) — https://github.com/broadinstitute/wot
- **scanpy** (Recommended pre-processing of scRNA-seq data before process_data)
- **scikit-learn** (Provides the PCA fit stored in data_pt['pca'] for dimensionality reduction)
- **UMAP** (Provides the UMAP fit stored in data_pt['um'] for visualization of trajectories)

## Examples

```
prescient process_data && bash weinreb-interpolate.sh
```

## Evaluation signals

- Simulated day-4 populations from PRESCIENT achieve lower Wasserstein distance to the actual held-out day-4 cells than the distance to the actual day-2 and day-6 populations and than the WOT linearly interpolated baseline
- Five random seeds are trained and per-seed testing and training distances are both recorded in wasserstein_distances.csv
- The data_pt object contains all required fields: normalized expression, timepoint labels, cell types, gene features, PCA/UMAP embeddings, and pre-computed growth weights
- Training converges with day 6 treated as the steady-state terminal target, and the training distance against day 6 is tracked per seed

## Limitations

- The model requires the final time point of the dataset to be at steady-state
- Model performance depends on growth rates and would benefit from better estimates of proliferation
- Information is lost about individual genes when transforming data into PCA space, and lowly-expressed genes important to cell fate decisions may be dropped altogether

## Evidence

- [methods] The `prescient process_data` command will generate a torch pt file `data_pt` (serialized dictionary) that contains all the necessary information for downstream training, simulations, and: "The `prescient process_data` command will generate a torch pt file `data_pt` (serialized dictionary) that contains all the necessary information for downstream training, simulations, and"
- [readme] weinreb-interpolate.sh trains 5 seeds of 2 layer 400 unit models on only lineage tracing data with ground truth proliferation rates for the interpolation task: [evidence span withheld — anchor status: external_doc]
- [intro] evaluating PRESCIENT's ability to generate held-out timepoints and to predict cell fate bias, i.e. the probability a cell enters a particular fate given its initial state: "evaluating PRESCIENT’s ability to generate held-out timepoints and to predict cell fate bias, i.e. the probability a cell enters a particular fate given its initial state"
- [discussion] the model is subject to constraints and assumptions, for example requiring that the final time point of the dataset be at steady-state.: "the model is subject to constraints and assumptions, for example requiring that the final time point of the dataset be at steady-state."
