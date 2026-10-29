---
name: paired-nonparametric-hypothesis-testing
description: Use when two configurations of the same method (e.g. weighted vs unweighted
  decoupleR runs) yield paired scores over identical dataset/regulator combinations
  and a directional significance claim must be tested nonparametrically.
when_to_use_negative:
- Samples are unpaired or come from independent runs — the signed-rank test requires
  matched pairs of the same evaluation units (dataset/regulator combinations).
- You are comparing entirely different methods against each other rather than the
  same method with vs without weights — that is a different benchmark question in
  decoupleRBench.
- Paired differences are plausibly normal — a paired t-test would be the more appropriate
  parametric choice.
edam_operation: http://edamontology.org/operation_2262
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: decoupleR
  role: Provides the unified framework of activity-inference methods (weight-leveraging
    and weight-ignoring) whose scores are compared in the paired test
  repo: https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html
- name: decoupleRBench
  role: Produces the per-method perturbation benchmark scores (AUROC/AUPRC with downsampling)
    that form the paired samples fed to the Wilcoxon test
  repo: https://github.com/saezlab/decoupleRBench/blob/main/vignettes/how-to-bench.Rmd
- name: R
  role: stats::wilcox.test with alternative='greater' performs the one-sided paired
    signed-rank test; R scripts run_rna_bench.R and run_php_bench.R generate the benchmark
    inputs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbac016
    title: 'decoupleR: ensemble of computational methods to infer biological activities
      from omics data'
  source_content_hash: sha256:c3e76b85a65b61dac4425da4ba2290983f56062f6246c3a10336a614ff80d0a7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - decoupleR
  - decoupleRBench
  edam_operation: http://edamontology.org/operation_2262
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# paired-nonparametric-hypothesis-testing

## Summary

Applies a one-sided paired Wilcoxon signed-rank test (R stats::wilcox.test) to per-method paired performance deltas, e.g. decoupleR perturbed-regulator recovery scores computed with interaction weights versus without, to establish that weight-leveraging methods perform significantly better (P < 2.2e-16, Supplementary Fig. S4).

## When to use

Use when two configurations of the same method (e.g. weighted vs unweighted decoupleR runs) yield paired scores over identical dataset/regulator combinations and a directional significance claim must be tested nonparametrically.

## When NOT to use

- Samples are unpaired or come from independent runs — the signed-rank test requires matched pairs of the same evaluation units (dataset/regulator combinations).
- You are comparing entirely different methods against each other rather than the same method with vs without weights — that is a different benchmark question in decoupleRBench.
- Paired differences are plausibly normal — a paired t-test would be the more appropriate parametric choice.

## Inputs

- Per-method benchmark result tables from decoupleRBench for run_rna_bench.R (transcriptomic perturbation experiments)
- Per-method benchmark result tables from decoupleRBench for run_php_bench.R (phospho-proteomic perturbation experiments)
- Method partition: weight-leveraging methods and their weight-ignoring counterparts
- Paired scores per dataset/regulator combination (with-weights vs without-weights)

## Outputs

- weights_comparison.tsv (paired comparison table of with-weights vs without-weights scores and deltas)
- wilcoxon_weights_test.json (one-sided Wilcoxon signed-rank test statistic and P-value)

## How to apply

Load the per-method benchmark results from the transcriptomic (run_rna_bench.R) and phospho-proteomic (run_php_bench.R) perturbation benchmarks produced with decoupleRBench. Partition evaluated methods into weight-leveraging methods (those using the sign/weight of network interactions) and their weight-ignoring counterparts. Pair each weight-leveraging method's perturbed-regulator recovery score computed with weights against the same score computed without weights across all dataset/regulator combinations, then compute per-method deltas and summarize the paired differences. Run a one-sided Wilcoxon signed-rank test (R stats::wilcox.test, alternative='greater') on the paired differences and record the statistic and P-value. The pairing rationale comes from decoupleRBench's evaluation design, where all experiments are concatenated into a response vector (perturbed or not) and a predictor vector of regulator activities, with AUROC/AUPRC computed under a negative-class downsampling strategy; pairing the identical evaluation units is what justifies the signed-rank test.

## Related tools

- **decoupleR** (Provides the unified framework of activity-inference methods (weight-leveraging and weight-ignoring) whose scores are compared in the paired test) — https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html
- **decoupleRBench** (Produces the per-method perturbation benchmark scores (AUROC/AUPRC with downsampling) that form the paired samples fed to the Wilcoxon test) — https://github.com/saezlab/decoupleRBench/blob/main/vignettes/how-to-bench.Rmd
- **R** (stats::wilcox.test with alternative='greater' performs the one-sided paired signed-rank test; R scripts run_rna_bench.R and run_php_bench.R generate the benchmark inputs)

## Examples

```
wilcox.test(perf_with_weights, perf_without_weights, paired = TRUE, alternative = "greater")
```

## Evaluation signals

- Reproduced one-sided Wilcoxon signed-rank P-value is below 2.2e-16, matching Supplementary Fig. S4.
- Test is run with alternative='greater' and paired=TRUE on within-method deltas, so the test statistic is consistent with positive deltas favoring weight-leveraging methods.
- Paired samples have identical lengths and identical dataset/regulator keys on both sides of each pair.
- weights_comparison.tsv contains per-method paired scores and deltas; wilcoxon_weights_test.json contains a single statistic and P-value pair.
- Direction of the conclusion matches the paper's finding: weight-leveraging methods perform better when weights are taken into account.

## Limitations

- True positive classes are limited by the regulators covered in the perturbation experiments, so decoupleRBench uses a downsampling strategy (equal numbers of randomly sampled negatives per permutation), which introduces resampling variability into the paired scores.
- The reported P-value (<2.2e-16) is near the numerical floor of the test, so it functions as a strong directional confirmation rather than a calibrated effect size; report the paired deltas alongside the P-value.
- The conclusion is specific to the decoupleR benchmark settings (transcriptomic and phospho-proteomic perturbation experiments with OmniPath-derived prior knowledge); it does not automatically generalize to other scoring tasks.

## Evidence

- [supplementary] weight-leveraging significance: [evidence span withheld — anchor status: not_found]
- [readme] benchmark evaluation design: [evidence span withheld — anchor status: external_doc]
- [readme] downsampling rationale: [evidence span withheld — anchor status: external_doc]
- [readme] benchmark scripts: "# Run perturbation benchmark
Rscript R/process/run_rna_bench.R
Rscript R/process/run_php_bench.R"
- [abstract] decoupleR method scope: "including methods that leverage mode of regulation and weights of interactions, which are not present in other frameworks"
