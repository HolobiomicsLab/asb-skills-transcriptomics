---
name: binary-fate-classification-auroc-evaluation
description: Use when validating whether a trajectory/fate model's predicted fate-bias
  probabilities match observed clonal outcomes on a held-out progenitor set (e.g.
  the 335-cell Weinreb day-2 test set), where AUROC, Pearson r, and covered fraction
  are the acceptance metrics.
when_to_use_negative:
- The task is interpolation/generation of held-out time points rather than clonal
  fate-bias classification — the paper separates the Weinreb interpolation task (weinreb-interpolate.sh)
  from the fate task (weinreb-fate.sh).
- No lineage-tracing ground truth exists, so observed clonal fate bias cannot be computed
  and Pearson r/AUROC cannot be validated.
- The dataset's final time point is not at steady-state, violating the model assumption
  the fate-bias comparison rests on.
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0203
tools:
- name: PRESCIENT
  role: Trains potential-energy models and simulates trajectories from held-out progenitor
    states to produce fate-bias probabilities for scoring
  repo: https://github.com/gifford-lab/prescient
- name: scanpy
  role: Pre-processes the Weinreb lineage-tracing scRNA-seq matrix into .h5ad/.csv
    inputs with time-point labels and cell type annotations
- name: PyTorch
  role: Backend for training 2-layer 400-unit PRESCIENT models across seeds and proliferation
    regimes
- name: wot
  role: Optimal-transport baseline whose fate-bias predictions are scored alongside
    PRESCIENT
  repo: https://github.com/broadinstitute/wot
- name: prescient-analysis
  role: Provides the training bash script (weinreb-fate.sh) and 03{a-d}-weinreb2020-fate
    notebooks for the fate-bias evaluation workflow
  repo: https://github.com/gifford-lab/prescient-analysis
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-23518-w
    title: Generative modeling of single-cell time series with PRESCIENT enables prediction
      of cell trajectories with interventions
  source_content_hash: sha256:f360f2a1249061bf2f3cb9dd9ac1ed9bf1c12fe08f72a33a6cf15116389654c2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gifford-lab/prescient
  tools:
  - PRESCIENT
  - PyTorch
  - prescient-analysis
  - scanpy
  - scikit-learn
  - wot
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# binary-fate-classification-auroc-evaluation

## Summary

Scores predicted clonal fate bias of progenitor cells against held-out lineage-tracing ground truth using Pearson correlation, AUROC for binary fate classification, and the covered fraction of progenitors with a fate call, benchmarking a trajectory model (PRESCIENT) against WOT, FateID, and the clonal upper bound.

## When to use

Use when validating whether a trajectory/fate model's predicted fate-bias probabilities match observed clonal outcomes on a held-out progenitor set (e.g. the 335-cell Weinreb day-2 test set), where AUROC, Pearson r, and covered fraction are the acceptance metrics.

## When NOT to use

- The task is interpolation/generation of held-out time points rather than clonal fate-bias classification — the paper separates the Weinreb interpolation task (weinreb-interpolate.sh) from the fate task (weinreb-fate.sh).
- No lineage-tracing ground truth exists, so observed clonal fate bias cannot be computed and Pearson r/AUROC cannot be validated.
- The dataset's final time point is not at steady-state, violating the model assumption the fate-bias comparison rests on.

## Inputs

- Trained PRESCIENT models (data_pt torch objects built by `prescient process_data` with normalized expression, day-2/4/6 time-point labels, and cell type annotations)
- Held-out day-2 progenitor states (335-cell clonal test set) from the Weinreb et al. lineage-tracing dataset
- Observed clonal fate bias from lineage tracing (ground truth for correlation and AUROC)
- Baseline predictions from WOT (broadinstitute/wot) and FateID
- Held-out clonal upper-bound fate calls

## Outputs

- fate_bias_metrics.csv with method-by-method rows (Pearson r, AUROC, covered fraction per proliferation regime: no-proliferation, lineage-tracing growth rates, expression-derived growth rates)
- Per-method AUROC values for binary fate classification against the clonal upper bound

## How to apply

Train candidate models on the day-2/4/6 Weinreb hematopoiesis data (e.g. 5 seeds of 2-layer 400-unit PRESCIENT models per regime: growth weights zeroed, empirically measured lineage-tracing growth rates, and expression-derived growth rates). For each model, simulate trajectories from held-out day-2 progenitor states and compute clonal fate bias — the probability a cell enters a particular fate given its initial state — for the 335-cell test set. Score each method by Pearson correlation with observed fate bias, AUROC for fate classification, and the covered fraction of progenitors with a fate call, and compare against the WOT and FateID baselines and the held-out clonal upper bound. Accept the evaluation when the proliferation-aware variant improves over the no-proliferation run and approaches the paper's reported values (mean r = 0.347 ± 0.029, AUROC = 0.692 ± 0.012, per Fig. 2f–h).

## Related tools

- **PRESCIENT** (Trains potential-energy models and simulates trajectories from held-out progenitor states to produce fate-bias probabilities for scoring) — https://github.com/gifford-lab/prescient
- **scanpy** (Pre-processes the Weinreb lineage-tracing scRNA-seq matrix into .h5ad/.csv inputs with time-point labels and cell type annotations)
- **PyTorch** (Backend for training 2-layer 400-unit PRESCIENT models across seeds and proliferation regimes)
- **wot** (Optimal-transport baseline whose fate-bias predictions are scored alongside PRESCIENT) — https://github.com/broadinstitute/wot
- **prescient-analysis** (Provides the training bash script (weinreb-fate.sh) and 03{a-d}-weinreb2020-fate notebooks for the fate-bias evaluation workflow) — https://github.com/gifford-lab/prescient-analysis

## Examples

```
bash scripts/weinreb-fate.sh
```

## Evaluation signals

- fate_bias_metrics.csv exists with rows covering all PRESCIENT variants plus the WOT, FateID, and held-out clonal upper-bound baselines.
- Proliferation-aware PRESCIENT run achieves Pearson r ≈ 0.347 ± 0.029 and AUROC ≈ 0.692 ± 0.012, matching or approaching the values reported in Fig. 2f–h.
- The empirically-derived-proliferation variant improves over the no-proliferation (w zeroed) run on both Pearson r and AUROC.
- Each method row reports the covered fraction of progenitors with a fate call, so scores are interpreted against comparable coverage.
- Fate-bias predictions are computed only on the 335-cell held-out day-2 test set, never on training cells.

## Limitations

- The model requires the final time point of the dataset to be at steady-state.
- Performance depends on growth-rate estimates; the model improves when incorporating growth rates and would likely benefit from better estimates of proliferation.
- Information is lost about individual genes when transforming data into PCA space, and lowly-expressed genes important to cell fate decisions may be dropped altogether.
- Association of certain genes with specific cell fates does not necessarily imply causality, so high AUROC on this benchmark does not certify mechanistic correctness.

## Evidence

- [abstract] PRESCIENT is able to predict the fate biases of progenitor cells in hematopoiesis when accounting for cell proliferation, improving upon the best-performing existing method.: "PRESCIENT is able to predict the fate biases of progenitor cells in hematopoiesis when accounting for cell proliferation, improving upon the best-performing existing method."
- [intro] fate bias definition and evaluation goal: "evaluating PRESCIENT’s ability to generate held-out timepoints and to predict cell fate bias, i.e. the probability a cell enters a particular fate given its initial state"
- [readme] training configuration for the fate task: [evidence span withheld — anchor status: not_found]
- [methods] model inputs required before fate scoring: "For training, all that is needed is normalized gene expression, time-point labels, and cell type annotations."
