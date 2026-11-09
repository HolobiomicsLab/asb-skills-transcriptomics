---
name: network-weight-leveraging-method-comparison
description: Use when comparing enrichment/activity-inference methods that differ
  only in whether they leverage interaction weights, or when validating that weight-aware
  decoupleR methods outperform their weight-ignoring counterparts on perturbed-regulator
  prediction.
when_to_use_negative:
- When comparing methods that differ in more than weight handling (e.g. different
  network resources or statistics entirely) — this skill tests the specific with-weights
  vs. without-weights contrast for paired method counterparts.
- When no perturbation ground truth (perturbed regulators) is available for AUROC/AUPRC
  evaluation, since decoupleRBench requires a response vector of perturbed vs. non-perturbed
  regulators.
- When the network interactions carry no sign or weight information, since the contrast
  being tested does not exist.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3577
tools:
- name: decoupleR
  role: Unified framework whose weight-leveraging and weight-ignoring methods are
    benchmarked for perturbed-regulator prediction
  repo: https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html
- name: decoupleRBench
  role: R package that evaluates activity-inference method performance on perturbation
    experiments using AUROC/AUPRC with negative-class downsampling
  repo: https://github.com/saezlab/decoupleRBench
- name: R
  role: Language in which the benchmark scripts and wilcox.test paired analysis are
    run
- name: OmniPath
  role: Meta-resource of prior-knowledge networks (>100 databases) supplying the signed,
    weighted interactions used in the comparison
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbac016
    title: 'decoupleR: ensemble of computational methods to infer biological activities
      from omics data'
  source_content_hash: sha256:2b4154628f921492d65f1e1e1328c30a36e1a17541b4882174567cc1ad325d45
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - OmniPath
  - R
  - decoupleR
  - decoupleRBench
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# network-weight-leveraging-method-comparison

## Summary

Reproduces the decoupleR benchmark analysis testing whether regulator-activity inference methods perform better when the sign/weight of prior-knowledge network interactions is taken into account versus ignored, using paired one-sided Wilcoxon signed-rank testing across dataset/regulator combinations.

## When to use

Use when comparing enrichment/activity-inference methods that differ only in whether they leverage interaction weights, or when validating that weight-aware decoupleR methods outperform their weight-ignoring counterparts on perturbed-regulator prediction.

## When NOT to use

- When comparing methods that differ in more than weight handling (e.g. different network resources or statistics entirely) — this skill tests the specific with-weights vs. without-weights contrast for paired method counterparts.
- When no perturbation ground truth (perturbed regulators) is available for AUROC/AUPRC evaluation, since decoupleRBench requires a response vector of perturbed vs. non-perturbed regulators.
- When the network interactions carry no sign or weight information, since the contrast being tested does not exist.

## Inputs

- decoupleRBench benchmark results from Rscript R/process/run_rna_bench.R (transcriptomic perturbation experiments)
- decoupleRBench benchmark results from Rscript R/process/run_php_bench.R (phospho-proteomic perturbation experiments)
- Prior-knowledge network with signed, weighted interactions (e.g. from OmniPath meta-resource)
- Benchmark data as a count table with a corresponding metadata table

## Outputs

- weights_comparison.tsv — paired per-method comparison table (with-weights vs. without-weights scores across dataset/regulator combinations)
- wilcoxon_weights_test.json — one-sided Wilcoxon signed-rank statistic and P-value

## How to apply

Load per-method benchmark outputs from the decoupleRBench transcriptomic (run_rna_bench.R) and phospho-proteomic (run_php_bench.R) perturbation pipelines. Partition evaluated methods into weight-leveraging methods (those using the sign and weight of network interactions) and their weight-ignoring counterparts, then pair each method's perturbed-regulator recovery score computed with weights against the same score computed without weights across all dataset/regulator combinations. Compute per-method performance deltas and summarize the paired differences. Run a one-sided Wilcoxon signed-rank test (R stats::wilcox.test with alternative='greater') on the paired differences; the article reports P < 2.2e-16 (Supplementary Fig. S4), so weight-leveraging methods should significantly outperform when weights are used. Evaluation in decoupleRBench concatenates all experiments into a response vector (regulator perturbed or not) and a predictor vector (regulator activities), computing AUROC and AUPRC with a downsampling strategy that samples an equal number of negative classes per permutation because true positive classes are limited by regulators covered in perturbation experiments.

## Related tools

- **decoupleR** (Unified framework whose weight-leveraging and weight-ignoring methods are benchmarked for perturbed-regulator prediction) — https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html
- **decoupleRBench** (R package that evaluates activity-inference method performance on perturbation experiments using AUROC/AUPRC with negative-class downsampling) — https://github.com/saezlab/decoupleRBench
- **R** (Language in which the benchmark scripts and wilcox.test paired analysis are run)
- **OmniPath** (Meta-resource of prior-knowledge networks (>100 databases) supplying the signed, weighted interactions used in the comparison)

## Examples

```
Rscript R/process/run_rna_bench.R && Rscript R/process/run_php_bench.R
```

## Evaluation signals

- Reproduced one-sided Wilcoxon signed-rank P-value is below 2.2e-16, matching the reported Supplementary Fig. S4 result
- The paired-difference distribution is predominantly positive, confirming weight-leveraging methods perform better when weights are taken into account
- weights_comparison.tsv contains one paired row per dataset/regulator combination for every weight-leveraging/weight-ignoring method pair
- wilcoxon_tests.json contains both the test statistic and the P-value from stats::wilcox.test(alternative='greater')
- Benchmark scores are AUROC/AUPRC values produced via decoupleRBench's downsampling strategy with equal numbers of sampled negative classes per permutation

## Limitations

- True positive classes are limited by regulators covered in perturbation experiments, hence decoupleRBench relies on downsampling of negative classes rather than using all negatives
- The reported result is specific to the transcriptomic and phospho-proteomic perturbation benchmarks evaluated in the decoupleR manuscript; performance may differ on other omics modalities or networks
- The weight-vs-no-weight contrast requires methods that have both weight-leveraging and weight-ignoring variants; methods without such counterparts cannot be paired

## Evidence

- [results] Methods that leverage interaction weights perform significantly better when those weights are taken into account (one-sided Wilcoxon signed-rank test, P-value <2.2e-16; Supplementary Fig. S4).: [evidence span withheld — anchor status: not_found]
- [abstract] decoupleR supports methods that leverage mode of regulation and weights of interactions.: "including methods that leverage mode of regulation and weights of interactions, which are not present in other frameworks"
- [readme] decoupleRBench evaluation protocol.: [evidence span withheld — anchor status: external_doc]
- [readme] Benchmark scripts location.: "Rscript R/process/run_rna_bench.R
Rscript R/process/run_php_bench.R"
- [abstract] Methods were evaluated on transcriptomic and phospho-proteomic perturbation experiments.: "Using decoupleR, we evaluated the performance of methods on transcriptomic and phospho-proteomic perturbation experiments."
