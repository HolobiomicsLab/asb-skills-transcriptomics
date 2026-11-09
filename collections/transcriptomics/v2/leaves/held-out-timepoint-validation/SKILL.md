---
name: held-out-timepoint-validation
description: Use when a longitudinal scRNA-seq dataset has ≥3 time points and you
  need to test whether a trained trajectory model can generate an unobserved intermediate
  population, comparing against an optimal-transport baseline.
when_to_use_negative:
- The dataset's final time point cannot be assumed to be at steady-state, since the
  PRESCIENT model requires it as the terminal target
- The task is fate-bias/clonal prediction rather than interpolation — use the `weinreb-fate.sh`-style
  setup with estimated proliferation rates instead
- Reliable proliferation/growth-rate estimates are unavailable, since model performance
  depends on growth rates
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: PRESCIENT
  role: Trains the potential model on days 2 and 6 and simulates the held-out day-4
    marginal population
  repo: https://github.com/gifford-lab/prescient
- name: prescient
  role: PyPI CLI; `prescient process_data` builds the data_pt torch object used for
    training and simulation
  repo: https://github.com/gifford-lab/prescient
- name: PyTorch
  role: Framework for the PRESCIENT potential model (2-layer 400-unit models) and
    rolling cells forward under the learned potential
- name: scanpy
  role: Recommended pre-processing of the scRNA-seq data before PRESCIENT training
- name: scikit-learn
  role: Provides the sklearn.decomposition.PCA object fit to normalized expression
    stored in data_pt
- name: UMAP
  role: umap.UMAP object fit to PCs used to produce UMAP dimensions in data_pt
- name: wot
  role: Generates the linearly interpolated day-4 population baseline for the Fig.
    2a Wasserstein distance comparison
  repo: https://github.com/broadinstitute/wot
- name: prescient-analysis
  role: Provides `weinreb-interpolate.sh` (trains 5 seeds) and notebooks 02a-e for
    the Weinreb interpolation workflow
  repo: https://github.com/gifford-lab/prescient-analysis
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-23518-w
    title: Generative modeling of single-cell time series with PRESCIENT enables prediction
      of cell trajectories with interventions
  source_content_hash: sha256:2de2d1ebea5aafd298def7d00aac7abbe1c7bf6876eb997cee1ecb14a7cd5f5d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gifford-lab/prescient
  tools:
  - PRESCIENT
  - PyTorch
  - UMAP
  - prescient
  - prescient-analysis
  - scanpy
  - scikit-learn
  - wot
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# held-out-timepoint-validation

## Summary

Validates a PRESCIENT generative model by holding out one time point (day 4 of the Weinreb mouse hematopoiesis lineage tracing dataset), training only on days 2 and 6, simulating the held-out marginal population, and scoring it by Wasserstein distance against both the true held-out cells and a WOT linear-interpolation baseline.

## When to use

Use when a longitudinal scRNA-seq dataset has ≥3 time points and you need to test whether a trained trajectory model can generate an unobserved intermediate population, comparing against an optimal-transport baseline.

## When NOT to use

- The dataset's final time point cannot be assumed to be at steady-state, since the PRESCIENT model requires it as the terminal target
- The task is fate-bias/clonal prediction rather than interpolation — use the `weinreb-fate.sh`-style setup with estimated proliferation rates instead
- Reliable proliferation/growth-rate estimates are unavailable, since model performance depends on growth rates

## Inputs

- Weinreb et al. mouse hematopoiesis lineage tracing scRNA-seq data for days 2, 4, and 6 (.h5ad/.csv formats accepted by PRESCIENT)
- Ground truth proliferation rates for the Weinreb lineage tracing data
- data_pt torch object from `prescient process_data` (normalized expression, time-point labels, cell type annotations, sklearn PCA, umap.UMAP, growth weights)
- WOT (broadinstitute/wot) interpolation configuration for the linear day-4 baseline

## Outputs

- Per-seed simulated day-4 marginal cell populations
- wasserstein_distances.csv containing per-seed testing (day-4) and training (day-6) distances plus WOT baseline distances
- Fig. 2a-style comparison of Wasserstein distances (PRESCIENT vs. WOT interpolation vs. day-2/day-6 distances)

## How to apply

Load days 2, 4, and 6 of the Weinreb scRNA-seq data and run `prescient process_data` to build the `data_pt` torch object containing normalized expression, timepoint labels, cell types, PCA/UMAP embeddings, and pre-computed growth weights. Train the PRESCIENT potential model (PyTorch, 2-layer 400-unit networks) on the day-2 population with day 6 as the steady-state terminal target, deliberately excluding day 4, repeated over 5 random seeds as in `weinreb-interpolate.sh`. For each seed, simulate the day-4 marginal population by rolling day-2 cells forward under the learned potential. Compute Wasserstein testing distances between each simulated day-4 population and the true held-out day-4 cells, plus the training distance against day 6. Generate the linearly interpolated day-4 baseline population with WOT (broadinstitute/wot) and aggregate per-seed testing/training distances with the WOT baseline values into `wasserstein_distances.csv` (Fig. 2a comparison). The model is considered validated if simulated day-4 populations achieve lower Wasserstein distance to the true day-4 cells than the WOT interpolation and than distances to day-2/day-6 populations.

## Related tools

- **PRESCIENT** (Trains the potential model on days 2 and 6 and simulates the held-out day-4 marginal population) — https://github.com/gifford-lab/prescient
- **prescient** (PyPI CLI; `prescient process_data` builds the data_pt torch object used for training and simulation) — https://github.com/gifford-lab/prescient
- **PyTorch** (Framework for the PRESCIENT potential model (2-layer 400-unit models) and rolling cells forward under the learned potential)
- **scanpy** (Recommended pre-processing of the scRNA-seq data before PRESCIENT training)
- **scikit-learn** (Provides the sklearn.decomposition.PCA object fit to normalized expression stored in data_pt)
- **UMAP** (umap.UMAP object fit to PCs used to produce UMAP dimensions in data_pt)
- **wot** (Generates the linearly interpolated day-4 population baseline for the Fig. 2a Wasserstein distance comparison) — https://github.com/broadinstitute/wot
- **prescient-analysis** (Provides `weinreb-interpolate.sh` (trains 5 seeds) and notebooks 02a-e for the Weinreb interpolation workflow) — https://github.com/gifford-lab/prescient-analysis

## Examples

```
bash scripts/weinreb-interpolate.sh
```

## Evaluation signals

- Simulated day-4 populations from each of the 5 seeds achieve a lower Wasserstein distance to the true day-4 cells than the WOT linear-interpolation baseline
- Per-seed testing distance to day 4 is lower than the distances between simulated day-4 populations and the actual day-2 and day-6 populations
- 5 independent seeds were trained and their testing/training distances are recorded alongside WOT baseline values in wasserstein_distances.csv
- data_pt contains the expected fields: normalized expression, timepoint labels, cell types, gene features, PCA and UMAP embeddings, and pre-computed growth weights

## Limitations

- The model requires the final time point of the dataset to be at steady-state, which constrains which datasets and holdout schemes are valid
- Model performance depends on growth rates and would benefit from better estimates of proliferation
- Information is lost about individual genes when transforming data into PCA space, and lowly-expressed genes important to cell fate decisions may be dropped altogether

## Evidence

- [methods] The `prescient process_data` command will generate a torch pt file `data_pt` (serialized dictionary): "The `prescient process_data` command will generate a torch pt file `data_pt` (serialized dictionary)"
- [intro] evaluating PRESCIENT's ability to generate held-out timepoints: "evaluating PRESCIENT’s ability to generate held-out timepoints and to predict cell fate bias, i.e. the probability a cell enters a particular fate given its initial state"
- [readme] weinreb-interpolate.sh trains 5 seeds: [evidence span withheld — anchor status: external_doc]
- [readme] interpolation notebooks on the Weinreb dataset: [evidence span withheld — anchor status: external_doc]
- [discussion] steady-state constraint on final time point: "the model is subject to constraints and assumptions, for example requiring that the final time point of the dataset be at steady-state."
