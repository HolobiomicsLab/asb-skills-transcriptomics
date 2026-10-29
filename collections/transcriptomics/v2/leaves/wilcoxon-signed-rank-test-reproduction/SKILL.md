---
name: wilcoxon-signed-rank-test-reproduction
description: Use when reproducing the decoupleR perturbation-benchmark weights ablation,
  i.e. when paired per-method with-weights vs without-weights perturbed-regulator
  recovery scores are available and a one-sided Wilcoxon signed-rank test must be
  run on the paired deltas.
when_to_use_negative:
- Scores are unpaired (e.g. comparing disjoint method families or independent datasets)
  — the Wilcoxon signed-rank test requires matched with-weights/without-weights pairs
  per dataset/regulator combination.
- The goal is the manuscript's overall method ranking (e.g. simple linear models and
  the consensus score across top methods) rather than the Supplementary Fig. S4 weights
  ablation.
- No decoupleRBench outputs exist yet; the raw count table and metadata must first
  be processed through run_rna_bench.R / run_php_bench.R before the paired deltas
  can be tested.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0203
tools:
- name: decoupleRBench
  role: Produces the per-method perturbed-regulator recovery scores (AUROC/AUPRC with
    downsampled negative classes) that are paired for the weights comparison
  repo: https://github.com/saezlab/decoupleRBench
- name: decoupleR
  role: Supplies the weight-leveraging and weight-ignoring activity-inference methods
    whose benchmark scores are paired and tested
  repo: https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html
- name: R
  role: Runs stats::wilcox.test(alternative='greater') on the paired per-method deltas
    and writes the JSON result
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbac016
    title: 'decoupleR: ensemble of computational methods to infer biological activities
      from omics data'
  source_content_hash: sha256:59aeeb41861865749a57d0fefb288d950f9507a1ee5f95d0bd9794a4983d0b34
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saezlab/decoupleRBench
  tools:
  - R
  - decoupleR
  - decoupleRBench
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# wilcoxon-signed-rank-test-reproduction

## Summary

Reproduces the decoupleR manuscript's paired benchmark analysis showing that methods leveraging interaction weights perform significantly better when those weights are taken into account (one-sided Wilcoxon signed-rank test, P < 2.2e-16; Supplementary Fig. S4).

## When to use

Use when reproducing the decoupleR perturbation-benchmark weights ablation, i.e. when paired per-method with-weights vs without-weights perturbed-regulator recovery scores are available and a one-sided Wilcoxon signed-rank test must be run on the paired deltas.

## When NOT to use

- Scores are unpaired (e.g. comparing disjoint method families or independent datasets) — the Wilcoxon signed-rank test requires matched with-weights/without-weights pairs per dataset/regulator combination.
- The goal is the manuscript's overall method ranking (e.g. simple linear models and the consensus score across top methods) rather than the Supplementary Fig. S4 weights ablation.
- No decoupleRBench outputs exist yet; the raw count table and metadata must first be processed through run_rna_bench.R / run_php_bench.R before the paired deltas can be tested.

## Inputs

- Per-method benchmark results from the transcriptomic perturbation benchmark (produced by run_rna_bench.R with decoupleRBench)
- Per-method benchmark results from the phospho-proteomic perturbation benchmark (produced by run_php_bench.R with decoupleRBench)
- decoupleRBench input tibble of user-specified settings plus benchmark count table and corresponding metadata table
- Method metadata partitioning decoupleR methods into weight-leveraging (sign/weight-aware) and weight-ignoring counterparts

## Outputs

- weights_comparison.tsv — paired per-method with-weights vs without-weights recovery scores and performance deltas across dataset/regulator combinations
- wilcoxon_weights_test.json — one-sided Wilcoxon signed-rank test statistic and P-value (expected < 2.2e-16)

## How to apply

First produce the per-method benchmark results by running the manuscript's transcriptomic and phospho-proteomic perturbation benchmarks (Rscript R/process/run_rna_bench.R and Rscript R/process/run_php_bench.R) with decoupleRBench, which concatenates experiments into a response vector (regulator perturbed or not) and a predictor vector (regulator activities), then computes AUROC and AUPRC with a downsampling strategy over negative classes. Partition the evaluated methods into weight-leveraging methods (those using the sign/weight of network interactions) and their weight-ignoring counterparts. Pair each weight-leveraging method's recovery score computed with weights against the same score without weights across all dataset/regulator combinations, and compute per-method performance deltas on these paired differences. Run a one-sided Wilcoxon signed-rank test via R stats::wilcox.test with alternative='greater' on the paired differences and record the statistic and P-value; the manuscript reports P < 2.2e-16 in Supplementary Fig. S4. Write the paired comparison table to weights_comparison.tsv and the test statistic and P-value to wilcoxon_weights_test.json.

## Related tools

- **decoupleRBench** (Produces the per-method perturbed-regulator recovery scores (AUROC/AUPRC with downsampled negative classes) that are paired for the weights comparison) — https://github.com/saezlab/decoupleRBench
- **decoupleR** (Supplies the weight-leveraging and weight-ignoring activity-inference methods whose benchmark scores are paired and tested) — https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html
- **R** (Runs stats::wilcox.test(alternative='greater') on the paired per-method deltas and writes the JSON result)

## Examples

```
stats::wilcox.test(deltas, alternative = "greater")  # one-sided test on paired with- vs without-weights recovery deltas
```

## Evaluation signals

- Reproduced one-sided Wilcoxon signed-rank P-value is below 2.2e-16, matching Supplementary Fig. S4.
- Test is run with alternative='greater' and the direction of effect confirms weight-leveraging methods perform better with weights (positive paired deltas).
- Every weight-leveraging method has both a with-weights and a without-weights score for each dataset/regulator combination (no missing pairs).
- weights_comparison.tsv contains per-method performance deltas for all pairings; wilcoxon_weights_test.json records both the test statistic and the P-value.

## Limitations

- True positive classes are limited by the regulators covered in the perturbation experiments, so decoupleRBench uses a downsampling strategy (an equal number of negative classes randomly sampled per permutation); recovery scores therefore depend on this sampling.
- The companion noise analysis is separate and long-running (run_rna_noise.R ~6 hours, run_php_noise.R ~2 hours) and is outside this skill's scope.
- The P < 2.2e-16 validation threshold is specific to the manuscript's transcriptomic and phospho-proteomic perturbation benchmark data; other resources or settings may not reproduce it.

## Evidence

- [results] Methods that leverage interaction weights perform significantly better when those weights are taken into account (one-sided Wilcoxon signed-rank test, P-value <2.2e-16; Supplementary Fig. S4).: [evidence span withheld — anchor status: not_found]
- [abstract] including methods that leverage mode of regulation and weights of interactions, which are not present in other frameworks: "including methods that leverage mode of regulation and weights of interactions, which are not present in other frameworks"
- [readme] In this manuscript we have built a separate R package to benchmark decoupleR,
called decoupleRBench.: "In this manuscript we have built a separate R package to benchmark decoupleR,
called decoupleRBench."
- [readme] Then, using different thresholds we can calculate AUROC and AUPRC for each method.: [evidence span withheld — anchor status: external_doc]
