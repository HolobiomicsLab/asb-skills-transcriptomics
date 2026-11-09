---
name: wasserstein-metric-evaluation-of-simulated-populations
description: Use when a generative model simulates a held-out intermediate timepoint
  of a scRNA-seq time series (e.g., PRESCIENT day-4 rollforward from day 2) and you
  must quantify recovery of the true marginal population.
when_to_use_negative:
- The task is clonal fate bias prediction rather than held-out timepoint interpolation
  (use the weinreb-fate workflow instead of weinreb-interpolate evaluation).
- No held-out ground-truth timepoint exists, so there is no real population against
  which a Wasserstein testing distance can be computed.
- Input cells have not been jointly embedded (e.g., PCA space from `prescient process_data`),
  since Wasserstein distances between populations are only comparable in a shared
  feature space.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: PRESCIENT
  role: Generative model whose per-seed simulated day-4 populations are scored with
    the Wasserstein testing distance
  repo: https://github.com/gifford-lab/prescient
- name: prescient
  role: CLI package whose `prescient process_data` command builds the data_pt torch
    object consumed downstream for training and simulation
  repo: https://github.com/gifford-lab/prescient
- name: PyTorch
  role: Framework for training the PRESCIENT potential model and storing the data_pt
    torch object
- name: scikit-learn
  role: Provides the PCA (data_pt['pca']) defining the feature space in which distances
    are computed
- name: scanpy
  role: Recommended pre-processing of the scRNA-seq inputs (.csv, .tsv, .txt, .h5ad)
    before training and evaluation
- name: wot
  role: Generates the linearly interpolated day-4 population baseline scored for the
    Fig. 2a comparison
  repo: https://github.com/broadinstitute/wot
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-23518-w
    title: Generative modeling of single-cell time series with PRESCIENT enables prediction
      of cell trajectories with interventions
  source_content_hash: sha256:1f8b1bd95d7cb39e628e91d4123923db605b943e79137b64406432de11034d45
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gifford-lab/prescient
  tools:
  - PRESCIENT
  - PyTorch
  - prescient
  - scanpy
  - scikit-learn
  - wot
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# wasserstein-metric-evaluation-of-simulated-populations

## Summary

Evaluate simulated single-cell timepoint populations (e.g., PRESCIENT day-4 rollouts on the Weinreb hematopoiesis dataset) by computing Wasserstein testing distances against held-out ground-truth cells and comparing against baselines such as the WOT linearly interpolated population.

## When to use

Use when a generative model simulates a held-out intermediate timepoint of a scRNA-seq time series (e.g., PRESCIENT day-4 rollforward from day 2) and you must quantify recovery of the true marginal population.

## When NOT to use

- The task is clonal fate bias prediction rather than held-out timepoint interpolation (use the weinreb-fate workflow instead of weinreb-interpolate evaluation).
- No held-out ground-truth timepoint exists, so there is no real population against which a Wasserstein testing distance can be computed.
- Input cells have not been jointly embedded (e.g., PCA space from `prescient process_data`), since Wasserstein distances between populations are only comparable in a shared feature space.

## Inputs

- Simulated day-4 marginal populations generated per seed by rolling PRESCIENT-trained cells forward from day 2
- Held-out real day-4 scRNA-seq cells from the Weinreb et al. mouse hematopoiesis lineage tracing dataset
- Day-2 and day-6 populations (baseline distance references) and the day-6 steady-state terminal target used in training
- WOT linearly interpolated day-4 population baseline
- data_pt torch object produced by `prescient process_data` (normalized expression, timepoint labels, cell types, PCA/UMAP embeddings, growth weights)

## Outputs

- Per-seed Wasserstein testing distances of simulated day-4 populations vs. held-out day-4 cells
- Per-seed training Wasserstein distances against day 6
- WOT baseline day-4 Wasserstein distances and distances of simulated populations to day-2/day-6 references (Fig. 2a comparison values)
- wasserstein_distances.csv aggregating per-seed testing/training distances with WOT baseline values

## How to apply

For each trained model seed, roll cells forward from the day-2 Weinreb hematopoiesis population under the learned PRESCIENT potential to produce a simulated day-4 marginal population in the same PCA space as the real cells. Compute the Wasserstein testing distance between each simulated day-4 population and the held-out day-4 cells. Also compute the training distance of each seed's model against the day-6 steady-state terminal target to contextualize model fit. Generate the WOT (broadinstitute/wot) linearly interpolated day-4 population baseline and compute its day-4 distance for the Fig. 2a comparison, together with the distances of the simulated populations to the actual day-2 and day-6 populations. Aggregate the per-seed testing/training distances with WOT baseline values into a single wasserstein_distances.csv table for plotting and comparison.

## Related tools

- **PRESCIENT** (Generative model whose per-seed simulated day-4 populations are scored with the Wasserstein testing distance) — https://github.com/gifford-lab/prescient
- **prescient** (CLI package whose `prescient process_data` command builds the data_pt torch object consumed downstream for training and simulation) — https://github.com/gifford-lab/prescient
- **PyTorch** (Framework for training the PRESCIENT potential model and storing the data_pt torch object)
- **scikit-learn** (Provides the PCA (data_pt['pca']) defining the feature space in which distances are computed)
- **scanpy** (Recommended pre-processing of the scRNA-seq inputs (.csv, .tsv, .txt, .h5ad) before training and evaluation)
- **wot** (Generates the linearly interpolated day-4 population baseline scored for the Fig. 2a comparison) — https://github.com/broadinstitute/wot

## Examples

```
bash weinreb-interpolate.sh  # trains 5 seeds; then compute Wasserstein testing distances of simulated day-4 populations vs. held-out day-4 cells and write wasserstein_distances.csv
```

## Evaluation signals

- Simulated day-4 Wasserstein testing distances should outperform the WOT linearly interpolated baseline for the same held-out day-4 cells.
- Distances are reproducible across 5 random seeds (the weinreb-interpolate.sh training setup trains 5 seeds of 2-layer 400-unit models).
- Each seed's training distance against the day-6 steady-state target is recorded alongside its testing distance, so testing distances should be interpreted relative to training distances.
- Aggregation step must produce a well-formed wasserstein_distances.csv containing per-seed testing and training distances plus WOT baseline values.
- Simulated populations should also be scored against the actual day-2 and day-6 populations as reference distances.

## Limitations

- The PRESCIENT model requires the final time point of the dataset to be at steady-state, constraining which day-6 targets are valid.
- Model performance depends on growth rates and would benefit from better estimates of proliferation; evaluation may reflect growth-weighting quality.
- Information about individual genes is lost in the PCA transformation, so Wasserstein distances are computed in a reduced space that may miss lowly-expressed fate-relevant genes.

## Evidence

- [methods] For each seed, simulate the day-4 marginal population by rolling cells forward from day 2 under the learned potential.: [evidence span withheld — anchor status: not_found]
- [methods] Compute the Wasserstein testing distance between each simulated day-4 population and the held-out day-4 cells: [evidence span withheld — anchor status: not_found]
- [readme] weinreb-interpolate.sh trains 5 seeds of 2 layer 400 unit models on only lineage tracing data with ground truth proliferation rates for the interpolation task: [evidence span withheld — anchor status: external_doc]
- [results] Simulated day-4 populations from PRESCIENT outperform baselines, including their distance to the actual day-2 and day-6 populations and a linearly interpolated population predicted by WOT.: [evidence span withheld — anchor status: not_found]
- [methods] The `prescient process_data` command will generate a torch pt file `data_pt` (serialized dictionary): "The `prescient process_data` command will generate a torch pt file `data_pt` (serialized dictionary) that contains all the necessary information for downstream training, simulations, and"
