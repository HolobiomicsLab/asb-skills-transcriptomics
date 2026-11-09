---
name: held-out-timepoint-benchmark-comparison
description: Use when a trained trajectory model must be validated on held-out progenitor
  states and ranked against existing methods on clonal fate-bias prediction metrics.
when_to_use_negative:
- Data is a single time point with no held-out timepoint or clonal labels, so no benchmark
  against observed fate bias is possible
- The dataset's final time point is not at steady-state, violating the model's stated
  assumption and invalidating a fair fate-bias comparison
- The task is perturbation simulation or interpolation rather than clonal fate-bias
  benchmarking against WOT/FateID
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0203
tools:
- name: PRESCIENT
  role: Generative potential-energy model whose simulated trajectories produce the
    fate-bias predictions being benchmarked
  repo: github.com/gifford-lab/prescient
- name: WOT
  role: External baseline method scored for comparison on the same fate-bias metrics
  repo: github.com/broadinstitute/wot
- name: prescient-analysis
  role: Contains weinreb-fate.sh training scripts and fate-analysis notebooks used
    to generate the benchmark predictions
  repo: github.com/gifford-lab/prescient-analysis
- name: scanpy
  role: Used to pre-process the Weinreb et al. scRNA-seq matrix into the .h5ad/.csv
    inputs that feed the benchmarked models
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-23518-w
    title: Generative modeling of single-cell time series with PRESCIENT enables prediction
      of cell trajectories with interventions
  source_content_hash: sha256:42fcb3da0ac78510251d9eb7e8d90a11cce9b5e36a332e84178adf508af3f1d1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PRESCIENT
  - PyTorch
  - WOT
  - prescient-analysis
  - scanpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# held-out-timepoint-benchmark-comparison

## Summary

Benchmark a trajectory/fate-prediction model (PRESCIENT) against external baselines (WOT, FateID) and a held-out clonal upper bound by scoring fate-bias predictions with Pearson r, AUROC, and covered fraction, writing the method-by-method table to fate_bias_metrics.csv.

## When to use

Use when a trained trajectory model must be validated on held-out progenitor states and ranked against existing methods on clonal fate-bias prediction metrics.

## When NOT to use

- Data is a single time point with no held-out timepoint or clonal labels, so no benchmark against observed fate bias is possible
- The dataset's final time point is not at steady-state, violating the model's stated assumption and invalidating a fair fate-bias comparison
- The task is perturbation simulation or interpolation rather than clonal fate-bias benchmarking against WOT/FateID

## Inputs

- Simulated fate-bias predictions per method (probability a cell enters each fate given its initial state) for the 335 held-out day-2 progenitor cells
- Observed clonal fate bias on the Weinreb et al. lineage-tracing dataset
- Trained PRESCIENT models under no-proliferation, lineage-tracing growth-rate, and gene-expression-derived growth-rate regimes (growth tensor w)
- WOT and FateID baseline fate predictions

## Outputs

- fate_bias_metrics.csv with rows for all PRESCIENT variants and the WOT/FateID/upper-bound baselines (Pearson r, AUROC, covered fraction)
- Per-method AUROC for fate classification and covered fraction of progenitors with a fate call

## How to apply

Train the PRESCIENT potential-energy model on days 2/4/6 of the Weinreb hematopoiesis lineage-tracing data under three proliferation regimes (growth tensor w zeroed, empirically measured lineage-tracing growth rates, and gene-expression-derived growth rates). For each trained model, simulate trajectories from held-out day-2 progenitor states and compute clonal fate bias — the probability a cell enters a particular fate given its initial state — for the 335-cell test set. Score every method (each PRESCIENT variant, WOT, FateID, and the held-out clonal upper bound) by Pearson correlation with observed fate bias, AUROC for fate classification, and the covered fraction of progenitors receiving a fate call. The decision point is whether the proliferation-aware variant outperforms the no-proliferation run and the baselines; in the paper this was mean r = 0.347 ± 0.029 and AUROC = 0.692 ± 0.012. Write the per-method metric table to fate_bias_metrics.csv as the deliverable.

## Related tools

- **PRESCIENT** (Generative potential-energy model whose simulated trajectories produce the fate-bias predictions being benchmarked) — github.com/gifford-lab/prescient
- **WOT** (External baseline method scored for comparison on the same fate-bias metrics) — github.com/broadinstitute/wot
- **prescient-analysis** (Contains weinreb-fate.sh training scripts and fate-analysis notebooks used to generate the benchmark predictions) — github.com/gifford-lab/prescient-analysis
- **scanpy** (Used to pre-process the Weinreb et al. scRNA-seq matrix into the .h5ad/.csv inputs that feed the benchmarked models)

## Examples

```
bash scripts/weinreb-fate.sh
```

## Evaluation signals

- fate_bias_metrics.csv exists and contains rows for all PRESCIENT variants plus WOT, FateID, and the clonal upper bound
- The proliferation-aware PRESCIENT variant's Pearson r and AUROC match or approach the reported Fig. 2f-h values (mean r ≈ 0.347 ± 0.029, AUROC ≈ 0.692 ± 0.012)
- Proliferation-aware runs show measurable improvement over the no-proliferation (zeroed growth tensor w) run
- Covered fraction is reported per method so partial-coverage methods (e.g. FateID) are not silently compared against full-coverage predictions
- The 335-cell held-out day-2 progenitor test set is consistently used across all methods being scored

## Limitations

- The model requires the final time point of the dataset to be at steady-state, restricting which time-series can be benchmarked this way
- Performance depends on growth-rate estimates; better estimates of proliferation would likely further improve fate-bias scores
- Transforming expression into PCA space loses information about individual genes, and lowly-expressed genes important to fate decisions may be dropped from the scRNA-seq data

## Evidence

- [abstract] PRESCIENT is able to predict the fate biases of progenitor cells in hematopoiesis when accounting for cell proliferation, improving upon the best-performing existing method.: "PRESCIENT is able to predict the fate biases of progenitor cells in hematopoiesis when accounting for cell proliferation, improving upon the best-performing existing method."
- [intro] evaluating PRESCIENT's ability to generate held-out timepoints and predict cell fate bias: [evidence span withheld — anchor status: not_found]
- [methods] PRESCIENT takes as input longitudinal scRNA-seq data: "PRESCIENT takes as input longitudinal scRNA-seq data. For training, all that is needed is normalized gene expression, time-point labels, and cell type annotations."
- [readme] weinreb-fate.sh trains 5 seeds for clonal fate bias prediction: [evidence span withheld — anchor status: external_doc]
- [discussion] The model improves when incorporating growth rates: "The model also improves when incorporating growth rates, and would likely benefit from better estimates of proliferation."
