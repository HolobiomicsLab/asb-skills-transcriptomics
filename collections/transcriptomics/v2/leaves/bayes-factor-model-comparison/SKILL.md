---
name: bayes-factor-model-comparison
description: Use when comparing probabilistic phase-inference models (Tempo vs Cyclops,
  Cyclum, PCA) on single-cell RNA-seq data and you need a quantitative, per-cell-type
  measure of the evidence favoring one model's phase estimates over another's.
when_to_use_negative:
- Input is simulated scRNA-seq data matched to the method's own likelihood model rather
  than real droplet-based data — the article's motivation is that accuracy must be
  verified against real 10X Chromium data
- The methods being compared do not produce per-cell phase point estimates or a probabilistic
  model of expression over phase, so no Bayes factor can be defined between them
- The goal is cell-cycle phase ordering rather than circadian phase inference — existing
  methods were developed for cell cycle analysis and the ~20-gene core clock structure
  is what distinguishes this setting
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: Tempo
  role: Bayesian variational inference method whose phase model is the reference against
    which Bayes factors are computed; installed in the tempo conda environment (python
    >= 3.8, pytorch >= 1.9.0, scanpy >= 1.6, anndata, numpy, pandas, scikit-learn,
    scipy)
  repo: https://github.com/bauerbach95/tempo
- name: Cyclops
  role: Competing unsupervised phase-inference method benchmarked against Tempo in
    Table 1
- name: Cyclum
  role: Competing unsupervised phase-inference method benchmarked against Tempo in
    Table 1
- name: PCA
  role: Baseline phase-inference method applied with the core clock gene set and with
    all genes
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-34185-w
    title: 'Tempo: an unsupervised Bayesian algorithm for circadian phase inference
      in single-cell transcriptomics'
  source_content_hash: sha256:2b6b31b881f1939ab34f235d09df7d39059466ea74986b491296a2fc79314e5b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bauerbach95/tempo
  tools:
  - Cyclops
  - Cyclum
  - PCA
  - Tempo
  - anndata
  - numpy
  - pandas
  - scanpy
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# bayes-factor-model-comparison

## Summary

Bayes factor model comparison for circadian phase inference, benchmarking Tempo's Bayesian variational inference against competing unsupervised phase methods (Cyclops, Cyclum, PCA) on droplet-based scRNA-seq data. Bayes factors quantify the evidence that Tempo's phase model explains the data better than alternatives.

## When to use

Use when comparing probabilistic phase-inference models (Tempo vs Cyclops, Cyclum, PCA) on single-cell RNA-seq data and you need a quantitative, per-cell-type measure of the evidence favoring one model's phase estimates over another's.

## When NOT to use

- Input is simulated scRNA-seq data matched to the method's own likelihood model rather than real droplet-based data — the article's motivation is that accuracy must be verified against real 10X Chromium data
- The methods being compared do not produce per-cell phase point estimates or a probabilistic model of expression over phase, so no Bayes factor can be defined between them
- The goal is cell-cycle phase ordering rather than circadian phase inference — existing methods were developed for cell cycle analysis and the ~20-gene core clock structure is what distinguishes this setting

## Inputs

- anndata objects of droplet-based scRNA-seq data (10X Chromium mouse aorta light-dark dataset; Droin et al. hepatocyte time course), subset by cell type
- Per-cell phase estimates in hours from each method (Tempo, Cyclops, Cyclum, PCA with core clock genes and with all genes)
- Known sampling-time reference labels for each cell/cell type

## Outputs

- Per-cell-type median phase errors for each method (e.g., Tempo all-genes SMCs = 1.88 h)
- Log Bayes factors of Tempo vs each competing method per cell type
- A combined table written to phase_benchmark_results.csv

## How to apply

Run each phase-inference method — Tempo, Cyclops, Cyclum, and PCA (with the core clock gene set and with all genes) — on the same input data (e.g., 10X Chromium mouse aorta light-dark scRNA-seq subset to SMCs, fibroblasts, endothelial cells, and macrophages, or the Droin et al. hepatocyte time course) and record per-cell phase estimates in hours. For each method-cell-type combination, compute the median phase error against the known sampling-time reference, then compute the Bayes factor of Tempo against each competing method per cell type. Assemble the median phase errors and log Bayes factors into a single table (e.g., phase_benchmark_results.csv). Rationale: Tempo's Bayesian variational inference incorporates domain knowledge of the clock and quantifies phase estimation uncertainty, so the Bayes factor provides a calibrated comparison of evidence for its phase model against methods whose transcript likelihood distributions poorly approximate droplet-based generative distributions.

## Related tools

- **Tempo** (Bayesian variational inference method whose phase model is the reference against which Bayes factors are computed; installed in the tempo conda environment (python >= 3.8, pytorch >= 1.9.0, scanpy >= 1.6, anndata, numpy, pandas, scikit-learn, scipy)) — https://github.com/bauerbach95/tempo
- **Cyclops** (Competing unsupervised phase-inference method benchmarked against Tempo in Table 1)
- **Cyclum** (Competing unsupervised phase-inference method benchmarked against Tempo in Table 1)
- **PCA** (Baseline phase-inference method applied with the core clock gene set and with all genes)

## Examples

```
python run_test.py && python run_phase_benchmark.py --methods tempo cyclops cyclum pca --datasets aorta hepatocytes --out phase_benchmark_results.csv
```

## Evaluation signals

- Reproduced median phase errors match Table 1 (e.g., Tempo all-genes SMCs = 1.88 h on the aorta dataset)
- Bayes factors reproduce the reported orders of magnitude: 10^17896 for SMCs, 10^2257 for fibroblasts, 10^110 for endothelial cells, 10^184 for macrophages, and 10^1761 for hepatocytes
- Per-cell phase estimates are recorded in hours for every method-cell-type combination before computing errors
- run_test.py packaged smoke test prints 'SUCCESSFULLY FINISHED' before the benchmark is launched
- Median phase errors for Tempo are lower than Cyclops, Cyclum, and PCA across all cell types and both gene-set inputs

## Limitations

- Bayes factor magnitudes depend on the assumed transcript likelihood distribution; competing methods yield poor point estimates on droplet-based scRNA-seq because their likelihood choices poorly approximate the true generative distribution
- Tempo assumes CCG expression means follow 24 h sinusoidal patterns; analyses suggest the de novo cycler issue may stem from this assumption and more flexible waveform modeling is needed
- Explicit modeling of technical effects becomes necessary for data collected as multiple samples over time, which is not fully handled
- Subpopulation-specific gene amplitude variation is not explicitly modeled

## Evidence

- [results] Bayes factor magnitudes and Table 1 medians: [evidence span withheld — anchor status: not_found]
- [abstract] Accuracy of Tempo vs existing methods: "we demonstrate that Tempo yields more accurate estimates of circadian phase than existing methods"
- [abstract] Uncertainty quantification support for Bayesian comparison: "Tempo, a Bayesian variational inference approach that incorporates domain knowledge of the clock and quantifies phase estimation uncertainty"
- [intro] Likelihood distribution rationale on droplet data: "existing approaches yield poor point estimates of cell phase due to transcript likelihood distribution choices that do not closely approximate the true generative distribution"
- [readme] Environment and smoke-test prerequisite: "To test if Tempo works properly, run the run_test.py file using using the activated environment: python run_test.py"
- [methods] Dependency set for the benchmark environment: "Tempo requires python >= 3.8 and depends on the following python packages"
