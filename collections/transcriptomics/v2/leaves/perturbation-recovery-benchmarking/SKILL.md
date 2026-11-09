---
name: perturbation-recovery-benchmarking
description: Use when evaluating how well regulator (TF or kinase) activity inference
  methods recover perturbed regulators from transcriptomic or phospho-proteomic perturbation
  data, or when statistically comparing method variants such as weighted vs unweighted
  scoring.
when_to_use_negative:
- The goal is enrichment or activity inference on a single unperturbed omics dataset
  without a perturbation benchmark — use the decoupler methods directly instead of
  the benchmarking pipeline
- No ground truth exists about which regulators were perturbed, so AUROC/AUPRC recovery
  scoring and the downsampling of negative classes cannot be defined
- Comparing methods on simulated scaling data rather than perturbation experiments
  — the manuscript uses separate scalability scripts (get_simmulated_data.R, run_scale_bench.R)
  for that
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: decoupleR
  role: Infers per-method regulator activity estimates via the `decouple` wrapper,
    including weight-leveraging methods and the consensus mean z-score
  repo: https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html
- name: decoupleRBench
  role: R package that evaluates activity inference methods on perturbation experiments
    by building response/predictor vectors and computing AUROC/AUPRC with downsampling
  repo: https://github.com/saezlab/decoupleRBench
- name: R
  role: Runs the benchmark scripts and the one-sided Wilcoxon signed-rank test (stats::wilcox.test,
    alternative='greater') on paired per-method score differences
provenance:
  source_task_ids:
  - task_007
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbac016
    title: 'decoupleR: ensemble of computational methods to infer biological activities
      from omics data'
  source_content_hash: sha256:c241e5894dd7d0951cbe8d8ade3bd6527a099ec017dfc59ea2a4ee4579d8296d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - decoupleR
  - decoupleRBench
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# perturbation-recovery-benchmarking

## Summary

Benchmark decoupleR activity-inference methods on perturbation experiments by inferring per-regulator activities with decoupleRBench, scoring recovery of perturbed regulators via AUROC/AUPRC, and running paired statistical tests (e.g. one-sided Wilcoxon signed-rank) on method comparisons such as weighted vs unweighted modes or the mean z-score consensus.

## When to use

Use when evaluating how well regulator (TF or kinase) activity inference methods recover perturbed regulators from transcriptomic or phospho-proteomic perturbation data, or when statistically comparing method variants such as weighted vs unweighted scoring.

## When NOT to use

- The goal is enrichment or activity inference on a single unperturbed omics dataset without a perturbation benchmark — use the decoupler methods directly instead of the benchmarking pipeline
- No ground truth exists about which regulators were perturbed, so AUROC/AUPRC recovery scoring and the downsampling of negative classes cannot be defined
- Comparing methods on simulated scaling data rather than perturbation experiments — the manuscript uses separate scalability scripts (get_simmulated_data.R, run_scale_bench.R) for that

## Inputs

- Benchmark count table (transcriptomic or phospho-proteomic perturbation data from Zenodo record 5645208) plus a corresponding metadata table
- Prior knowledge network with interaction signs and weights (e.g. OmniPath-derived)
- Per-method regulator activity score matrices produced by the decoupleRBench pipeline
- Tibble of user-specified method settings for decoupleRBench

## Outputs

- Per-method AUROC and AUPRC recovery scores for perturbed regulators across dataset/regulator combinations
- Paired weight-vs-no-weight comparison table (weights_comparison.tsv)
- Wilcoxon signed-rank statistic and P-value (wilcoxon_weights_test.json)
- Consensus activity estimates per regulator (consensus_scores.csv) from mean z-score across methods

## How to apply

Run the perturbation benchmark pipeline with decoupleRBench (Rscript R/process/run_rna_bench.R for transcriptomics, Rscript R/process/run_php_bench.R for phospho-proteomics), which infers activities per regulator and experiment via the decoupleR `decouple` wrapper. For each method, concatenate all experiments into a response vector (regulator perturbed or not) and a predictor vector (regulator activities), then compute AUROC and AUPRC across thresholds using a downsampling strategy in which an equal number of negative classes are randomly sampled per permutation, since true positives are limited by regulators covered in perturbation experiments. For the weighted-vs-unweighted comparison, pair each weight-leveraging method's recovery score computed with weights against the same score without weights across dataset/regulator combinations, compute paired differences, and run a one-sided Wilcoxon signed-rank test (R stats::wilcox.test, alternative='greater'), expected to give P < 2.2e-16. For the consensus score, standardize each method's activity estimates to z-scores per regulator and average them, then verify the consensus (along with the multivariate and univariate linear models, mlm and ulm) ranks among the top methods for perturbed-regulator recovery.

## Related tools

- **decoupleR** (Infers per-method regulator activity estimates via the `decouple` wrapper, including weight-leveraging methods and the consensus mean z-score) — https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html
- **decoupleRBench** (R package that evaluates activity inference methods on perturbation experiments by building response/predictor vectors and computing AUROC/AUPRC with downsampling) — https://github.com/saezlab/decoupleRBench
- **R** (Runs the benchmark scripts and the one-sided Wilcoxon signed-rank test (stats::wilcox.test, alternative='greater') on paired per-method score differences)

## Examples

```
Rscript R/process/run_rna_bench.R
```

## Evaluation signals

- Reproduced one-sided Wilcoxon signed-rank P-value for weighted vs unweighted performance must be below 2.2e-16 (Supplementary Fig. S4), confirming weight-leveraging methods perform better with weights
- The consensus method must appear among the top-ranked methods for predicting perturbed regulators, alongside the multivariate (mlm) and univariate (ulm) linear models
- consensus_scores.csv must contain one consensus activity estimate per regulator with row identifiers matching the input per-method activity matrices
- AUROC/AUPRC curves must be computed with the downsampling scheme: an equal number of negative classes randomly sampled per permutation to balance limited true positives
- Benchmark runs must reproduce the manuscript's transcriptomic (run_rna_bench.R) and phospho-proteomic (run_php_bench.R) pipelines without mismatched experiment/regulator identifiers

## Limitations

- True positive classes are limited by the regulators covered in perturbation experiments, requiring a downsampling strategy for negative classes
- decoupleRBench builds on decoupleR and its `decouple` wrapper, so the R decoupleR package must be installed and the user should be familiar with its basics
- Noise and scalability analyses are slow: run_rna_noise.R takes ~6 hours and run_php_noise.R ~2 hours

## Evidence

- [abstract] Our findings suggest that simple linear models and the consensus score across top methods perform better than other methods at predicting perturbed regulators.: "Our findings suggest that simple linear models and the consensus score across top methods perform better than other methods at predicting perturbed regulators."
- [abstract] Using decoupleR, we evaluated the performance of methods on transcriptomic and phospho-proteomic perturbation experiments.: "Using decoupleR, we evaluated the performance of methods on transcriptomic and phospho-proteomic perturbation experiments."
- [readme] Then, using different thresholds we can calculate AUROC and AUPRC for each method.: [evidence span withheld — anchor status: external_doc]
- [readme] Downsampling strategy for limited negative classes in benchmark evaluation.: [evidence span withheld — anchor status: external_doc]
- [readme] decoupleRBench builds on decoupleR's decouple wrapper.: "`decoupleRBench` allows to evaluate the performance of biological activity 
inference methods using perturbation experiments. It builds on `decoupleR`, and 
more specifically the `decouple` wrapper"
