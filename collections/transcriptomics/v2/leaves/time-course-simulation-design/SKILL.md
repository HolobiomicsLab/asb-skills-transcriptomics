---
name: time-course-simulation-design
description: Use when benchmarking circadian or cell-cycle phase-inference methods
  on droplet-based scRNA-seq data and you need controlled simulations with ground-truth
  phases across library sizes, cell counts, and sampling conditions.
when_to_use_negative:
- Evaluating on real experimental scRNA-seq data where no ground-truth single-cell
  circadian phases exist — the theoretical optimum comparison is only possible under
  a known generative model
- Plate-based scRNA-seq studies where sparse-droplet library-size effects (median
  UMIs of 3000–20000) are not the concern being benchmarked
- Non-circadian oscillatory processes (e.g., cell cycle) where the ~20-gene core clock
  gene set and 24 h sinusoidal assumption do not apply
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Tempo
  role: Provides the Negative Binomial generative simulation utilities that produce
    ground-truth phase labels, and serves as the Bayesian variational inference method
    under benchmark
  repo: https://github.com/bauerbach95/tempo
- name: anndata
  role: Container format for simulated count matrices with per-cell ground-truth phase
    metadata
- name: numpy
  role: Numerical backend for generating simulated counts and computing phase errors
    and eCDFs
- name: pandas
  role: Tabulation of median phase errors per method and condition
- name: scanpy
  role: scRNA-seq preprocessing/analysis support on simulated datasets (scanpy>=1.6
    required by Tempo)
- name: PyTorch
  role: Backend for Tempo's Bayesian variational inference during benchmark runs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-34185-w
    title: 'Tempo: an unsupervised Bayesian algorithm for circadian phase inference
      in single-cell transcriptomics'
  source_content_hash: sha256:d95909d683b2cee2464d2803905a38cf6a77fb001ec5bbeee250e9353adab5b4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bauerbach95/tempo
  tools:
  - PyTorch
  - Python
  - Tempo
  - anndata
  - numpy
  - pandas
  - scanpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# time-course-simulation-design

## Summary

Design simulated circadian time-course scRNA-seq benchmark datasets with known ground-truth cell phases, generated from Tempo's Negative Binomial generative model, to quantify how closely a phase-inference method approaches the theoretical optimum.

## When to use

Use when benchmarking circadian or cell-cycle phase-inference methods on droplet-based scRNA-seq data and you need controlled simulations with ground-truth phases across library sizes, cell counts, and sampling conditions.

## When NOT to use

- Evaluating on real experimental scRNA-seq data where no ground-truth single-cell circadian phases exist — the theoretical optimum comparison is only possible under a known generative model
- Plate-based scRNA-seq studies where sparse-droplet library-size effects (median UMIs of 3000–20000) are not the concern being benchmarked
- Non-circadian oscillatory processes (e.g., cell cycle) where the ~20-gene core clock gene set and 24 h sinusoidal assumption do not apply

## Inputs

- Simulation grid specification (conditions: unsynchronized, ZT0/6/12/18; cell counts 500–5000; median UMI counts 3000–20000)
- Core clock gene set (~20 moderately expressed genes) used to restrict simulation and inference inputs
- Simulated Negative Binomial count matrices in anndata format with ground-truth circadian phases per cell

## Outputs

- Simulated scRNA-seq count matrices with known ground-truth phases
- Per-method absolute point-estimate phase errors for every method and condition
- Empirical cumulative distribution functions (eCDFs) of phase errors per method/condition (Fig. 2a/d-style panels) and tabulated median errors

## How to apply

Define a simulation grid spanning unsynchronized and ZT0/6/12/18 time-course conditions, cell counts from 500 to 5000, and median UMI counts from 3000 to 20000. Generate count matrices using the package's Negative Binomial simulation utilities (numpy/pandas/anndata) so each cell carries a known ground-truth circadian phase; this model was chosen because transcript likelihood distribution choices that poorly approximate the true generative distribution are exactly why existing methods fail on droplet-based data. Run the method under test (e.g., Tempo's Bayesian variational inference with pytorch backend) on both core-clock-gene-restricted and all-gene inputs, alongside baselines such as Cyclops, Cyclum, and PCA. Compute absolute point-estimate phase errors against the simulated ground truth and plot per-method error eCDFs to compare against the maximum-likelihood optimum under the true generative model.

## Related tools

- **Tempo** (Provides the Negative Binomial generative simulation utilities that produce ground-truth phase labels, and serves as the Bayesian variational inference method under benchmark) — https://github.com/bauerbach95/tempo
- **anndata** (Container format for simulated count matrices with per-cell ground-truth phase metadata)
- **numpy** (Numerical backend for generating simulated counts and computing phase errors and eCDFs)
- **pandas** (Tabulation of median phase errors per method and condition)
- **scanpy** (scRNA-seq preprocessing/analysis support on simulated datasets (scanpy>=1.6 required by Tempo))
- **PyTorch** (Backend for Tempo's Bayesian variational inference during benchmark runs)

## Examples

```
git clone https://github.com/bauerbach95/tempo && cd tempo && source install.sh && conda activate tempo && python run_test.py
```

## Evaluation signals

- Tempo's phase-error eCDF lies close to the theoretical optimum (maximum-likelihood phases under the true generative model), while Cyclops/Cyclum/PCA degrade at small library sizes
- The packaged validation check run_test.py must print 'SUCCESSFULLY FINISHED' before the pipeline is trusted
- Median absolute phase errors tabulated per method/condition reproduce the reported ordering (Tempo better than existing methods)
- Simulated datasets correctly span the grid: conditions {unsynchronized, ZT0, ZT6, ZT12, ZT18} × cell counts {500–5000} × median UMI {3000–20000}
- Phase errors are computed against the simulated ground-truth phases, so absolute error must be bounded in [0, 24 h]

## Limitations

- Simulations assume 24 h sinusoidal CCG expression means; the discussion notes the de novo cycler issue may stem from this assumption and more flexible waveform modeling is needed
- Subpopulation-specific gene amplitude variation is not explicitly modeled; a mapping from cell-state embedding to sinusoidal parameters would be ideal
- Technical effects for data collected as multiple samples over time are not explicitly modeled
- The benchmark uses only the ~20-gene core clock set plus all-gene variants; cell-type-specific CCG identities are often unknown in real data

## Evidence

- [other] Simulation grid and error metric: [evidence span withheld — anchor status: not_found]
- [other] Generative model and comparison baselines: [evidence span withheld — anchor status: not_found]
- [other] Expected result: [evidence span withheld — anchor status: not_found]
- [intro] Why the simulation model matters: "existing approaches yield poor point estimates of cell phase due to transcript likelihood distribution choices that do not closely approximate the true generative distribution"
- [readme] Validation gate: "Running the test should take less than 5 minutes. Successful completion of the test will yield a message "SUCCESSFULLY FINISHED"."
- [discussion] Sinusoidal assumption caveat: "this may be due, in part, to the assumption that the expression means of CCGs follow 24 h sinusoidal patterns across the circadian cycle"
