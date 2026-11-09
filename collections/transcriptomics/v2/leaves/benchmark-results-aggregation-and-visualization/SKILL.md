---
name: benchmark-results-aggregation-and-visualization
description: Use when a perturbation benchmark has produced per-regulator activity
  scores for multiple decoupleR methods and you must aggregate them into per-method
  performance metrics and ranking comparisons.
when_to_use_negative:
- You only need to infer regulator activities from a single dataset, not evaluate
  or rank inference methods — use decoupleR directly rather than the benchmark aggregation
  pipeline.
- Your perturbation data has no ground-truth labels of which regulators were perturbed,
  so no response vector for AUROC/AUPRC evaluation can be constructed.
- You are working with the Python scverse decoupler implementation for single-cell
  enrichment scoring, which is outside this R-based decoupleRBench pipeline.
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: decoupleR
  role: Infers regulator activities for each method via the 'decouple' wrapper; its
    methods are the objects being benchmarked and aggregated.
  repo: https://github.com/saezlab/decoupleR
- name: decoupleRBench
  role: Dedicated R package that runs the perturbation benchmark, concatenates experiments
    into response/predictor vectors, and computes AUROC/AUPRC per method with negative-class
    downsampling.
  repo: https://github.com/saezlab/decoupleRBench
- name: R
  role: Execution environment for the manuscript's benchmark scripts (get_data.R,
    run_php_bench.R) and output CSV generation.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbac016
    title: 'decoupleR: ensemble of computational methods to infer biological activities
      from omics data'
  source_content_hash: sha256:b0654a4d635c5f86a931c8319027a11e866d9077d3d0adc32564c53379f36f25
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/saezlab/decoupleR
  tools:
  - R
  - decoupleR
  - decoupleRBench
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# benchmark-results-aggregation-and-visualization

## Summary

Aggregates decoupleR regulator-activity scores from perturbation benchmarks (transcriptomic and phospho-proteomic) into per-method performance metrics (AUROC/AUPRC/sensitivity) and compares method rankings across omics modalities, in the decoupleR manuscript style.

## When to use

Use when a perturbation benchmark has produced per-regulator activity scores for multiple decoupleR methods and you must aggregate them into per-method performance metrics and ranking comparisons.

## When NOT to use

- You only need to infer regulator activities from a single dataset, not evaluate or rank inference methods — use decoupleR directly rather than the benchmark aggregation pipeline.
- Your perturbation data has no ground-truth labels of which regulators were perturbed, so no response vector for AUROC/AUPRC evaluation can be constructed.
- You are working with the Python scverse decoupler implementation for single-cell enrichment scoring, which is outside this R-based decoupleRBench pipeline.

## Inputs

- Perturbation benchmark data (count table plus metadata tibble) downloaded from Zenodo record 5645208
- Prior knowledge resource: kinase–substrate network (for the phospho-proteomics benchmark)
- Per-regulator activity scores inferred with the decoupleR 'decouple' wrapper across the 11 methods plus consensus score

## Outputs

- php_bench_performance.csv with per-method performance metrics (AUROC/AUPRC, sensitivity focus)
- Comparison of the phospho-proteomics method ranking against the transcriptomics ranking

## How to apply

First obtain the perturbation datasets and prior knowledge resources via 'Rscript R/process/get_data.R' (Zenodo record 5645208) and install the dedicated benchmarking package with devtools::install_github('saezlab/decoupleRBench'). Run the benchmark with 'Rscript R/process/run_php_bench.R', which applies the 11 decoupleR methods (AUCell, fast GSEA, GSVA, ORA, ULM, VIPER, etc.) plus the consensus score to the Hernandez-Armenta et al. 2017 phospho-proteomics perturbation collection against the kinase–substrate network. decoupleRBench builds a response vector (whether each regulator is perturbed) and a predictor vector (its inferred activity) by concatenating all experiments, then sweeps thresholds to compute AUROC and AUPRC; because true-positive classes are limited by regulators covered in perturbation experiments, negatives are downsampled so each permutation uses an equal number of randomly sampled negative classes. Aggregate these scores into per-method metrics (evaluation focused on sensitivity), write them to php_bench_performance.csv, and compare the phospho-proteomics method ranking against the transcriptomics ranking.

## Related tools

- **decoupleR** (Infers regulator activities for each method via the 'decouple' wrapper; its methods are the objects being benchmarked and aggregated.) — https://github.com/saezlab/decoupleR
- **decoupleRBench** (Dedicated R package that runs the perturbation benchmark, concatenates experiments into response/predictor vectors, and computes AUROC/AUPRC per method with negative-class downsampling.) — https://github.com/saezlab/decoupleRBench
- **R** (Execution environment for the manuscript's benchmark scripts (get_data.R, run_php_bench.R) and output CSV generation.)

## Examples

```
Rscript R/process/run_php_bench.R
```

## Evaluation signals

- php_bench_performance.csv is produced containing metrics covering all 11 decoupleR methods plus the consensus score.
- The phospho-proteomics sensitivity ranking reproduces the paper's top three methods: consensus, multivariate linear model and ULM.
- AUROC and AUPRC are computed at multiple thresholds with the downsampling strategy that samples an equal number of negative classes per permutation.
- The phospho-proteomics ranking is visually compared side-by-side against the transcriptomics (run_rna_bench.R) ranking.

## Limitations

- True-positive classes are limited by the regulators covered in the perturbation experiments, requiring a downsampling strategy for each permutation.
- The RNA noise analysis takes ~6 hours and the phospho-proteomics noise analysis ~2 hours; scalability analysis requires simulated data.
- The evaluation focuses on sensitivity, so rankings may not generalize to other metrics or perturbation collections (e.g. beyond Hernandez-Armenta et al. 2017).

## Evidence

- [readme] run_php_bench step: "# Run perturbation benchmark
Rscript R/process/run_rna_bench.R
Rscript R/process/run_php_bench.R"
- [readme] decoupleRBench evaluation strategy: [evidence span withheld — anchor status: external_doc]
- [abstract] top-performing methods finding: "Our findings suggest that simple linear models and the consensus score across top methods perform better than other methods at predicting perturbed regulators."
- [abstract] scope of evaluation: "Using decoupleR, we evaluated the performance of methods on transcriptomic and phospho-proteomic perturbation experiments."
