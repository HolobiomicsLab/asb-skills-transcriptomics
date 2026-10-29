---
name: jaccard-similarity-of-top-ranked-regulators
description: Use when comparing activity-inference methods (e.g. the 11 decoupleR
  methods) on the same omics benchmark to measure whether methods nominate the same
  top regulators, complementing median Spearman correlation of activity scores.
when_to_use_negative:
- Evaluating whether methods correctly predict perturbed regulators — that requires
  AUROC/AUPRC benchmarking via decoupleRBench, not set-overlap agreement.
- Input is a single method's activity scores with no second method to compare against
  (Jaccard requires pairwise method sets).
- Comparing regulator rankings across different prior-knowledge resources rather than
  across inference methods within one resource.
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: decoupleR
  role: Generates the regulator activity scores from the 11 inference methods whose
    top-5% regulator sets are compared; benchmarked via decoupleRBench
  repo: https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html
- name: R
  role: Language in which the benchmark scripts (run_rna_bench.R, run_php_bench.R)
    and Jaccard/Spearman agreement computation are executed
- name: Python
  role: Alternative implementation language for decoupleR activity scoring (decoupler-py)
  repo: https://github.com/saezlab/decoupler-py
- name: OmniPath
  role: Meta-resource of prior knowledge networks used to build the regulons underlying
    the activity estimates
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbac016
    title: 'decoupleR: ensemble of computational methods to infer biological activities
      from omics data'
  source_content_hash: sha256:436789162a20091306bcc7e310314a9b62eef62dbef828f92dd42a1077d1b12d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - OmniPath
  - Python
  - R
  - decoupleR
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Jaccard similarity of top-ranked regulators

## Summary

Quantifies cross-method agreement among decoupleR activity-inference methods by ranking regulators by inferred activity score, retaining the top 5% per method, and computing pairwise Jaccard indexes on those top-regulator sets (median reported: 0.23 for transcriptomics, 0.21 for phospho-proteomics).

## When to use

Use when comparing activity-inference methods (e.g. the 11 decoupleR methods) on the same omics benchmark to measure whether methods nominate the same top regulators, complementing median Spearman correlation of activity scores.

## When NOT to use

- Evaluating whether methods correctly predict perturbed regulators — that requires AUROC/AUPRC benchmarking via decoupleRBench, not set-overlap agreement.
- Input is a single method's activity scores with no second method to compare against (Jaccard requires pairwise method sets).
- Comparing regulator rankings across different prior-knowledge resources rather than across inference methods within one resource.

## Inputs

- Per-method regulator activity score matrices from the decoupleR transcriptomics benchmark (run_rna_bench.R)
- Per-method regulator activity score matrices from the phospho-proteomics benchmark (run_php_bench.R)
- Underlying benchmark data from Zenodo record 5645208

## Outputs

- method_agreement_metrics.csv with four statistics (median Spearman: transcriptomics and phospho-proteomics; median Jaccard: transcriptomics and phospho-proteomics)
- Optional Fig. 1B-style method-agreement heatmap

## How to apply

Load the per-method regulator activity score matrices produced by the perturbation benchmarks (run_rna_bench.R for transcriptomics, run_php_bench.R for phospho-proteomics, data from Zenodo record 5645208). For each method, rank regulators by activity score and retain the top 5% as the method's nominated regulator set. Compute all pairwise Jaccard indexes between the top-5% regulator sets of the 11 methods and take the median across method pairs; for the same omics collection also compute pairwise Spearman rank correlations of full activity scores and take their median (reported medians: Spearman 0.52 transcriptomics / 0.65 phospho-proteomics; Jaccard 0.23 / 0.21). Write the four statistics to method_agreement_metrics.csv and optionally render a Fig. 1B-style agreement heatmap.

## Related tools

- **decoupleR** (Generates the regulator activity scores from the 11 inference methods whose top-5% regulator sets are compared; benchmarked via decoupleRBench) — https://www.bioconductor.org/packages/release/bioc/html/decoupleR.html
- **R** (Language in which the benchmark scripts (run_rna_bench.R, run_php_bench.R) and Jaccard/Spearman agreement computation are executed)
- **Python** (Alternative implementation language for decoupleR activity scoring (decoupler-py)) — https://github.com/saezlab/decoupler-py
- **OmniPath** (Meta-resource of prior knowledge networks used to build the regulons underlying the activity estimates)

## Examples

```
Rscript R/process/run_rna_bench.R && Rscript R/process/run_php_bench.R
```

## Evaluation signals

- Median pairwise Spearman correlation across the 11 methods reproduces 0.52 for transcriptomics and 0.65 for phospho-proteomics.
- Median pairwise Jaccard index of top-5% regulator sets reproduces 0.23 (transcriptomics) and 0.21 (phospho-proteomics).
- method_agreement_metrics.csv contains exactly the four expected statistics.
- The top-5% retention filter is applied per method before Jaccard computation (rank-then-threshold order matters).

## Limitations

- Methods show differing activity distributions, so full score correlation and top-set overlap can disagree; agreement is 'general', with median Jaccard (~0.21–0.23) far below perfect overlap.
- Noise and scalability analyses are computationally expensive (~6 hours for RNA noise, ~2 hours for phospho-proteomics noise), so full reproduction requires substantial runtime.
- Analysis must be run per omics collection separately (transcriptomics vs phospho-proteomics) — medians are not pooled.

## Evidence

- [results] Cross-method agreement via Spearman and Jaccard: [evidence span withheld — anchor status: not_found]
- [methods] Top-5% ranking and Jaccard procedure: [evidence span withheld — anchor status: not_found]
- [full_text] Data provenance and method count: [evidence span withheld — anchor status: not_found]
- [readme] Benchmark pipeline entry point: "# Run perturbation benchmark
Rscript R/process/run_rna_bench.R
Rscript R/process/run_php_bench.R"
- [abstract] Methods evaluated on both omics: "Using decoupleR, we evaluated the performance of methods on transcriptomic and phospho-proteomic perturbation experiments."
