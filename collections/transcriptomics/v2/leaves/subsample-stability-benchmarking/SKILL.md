---
name: subsample-stability-benchmarking
description: Use when evaluating how a reference-gene weighting method's stability
  performance degrades at low qPCR sample sizes, e.g. reproducing a Figure 2C-style
  mean-SD-versus-n comparison via repeated random subsampling.
when_to_use_negative:
- The dataset is a normalized high-throughput expression matrix rather than raw CT
  values from a qPCR array, since weights and stability measures here are computed
  on CT values with ctVal_source = TRUE
- The goal is selecting the best RG combination for a single full-size experiment
  rather than characterizing performance as a function of sample size
- Aggregating more than 5 reference genes is intended, since geom(sd) does not have
  expected benefits over the regular geometric mean in that regime
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: InterOpt
  role: R package providing run_experiment and calcWeight to compute weighted RG2
    combinations, their aggregation weights, and SD/CV stability measures on subsampled
    CT data
  repo: https://github.com/asalimih/InterOpt
- name: devtools
  role: Used to install the InterOpt package from GitHub before running the benchmark
- name: R
  role: Execution environment for the subsampling loop, mc.cores parallelism, and
    aggregation of mean SD results
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2023.107945
    title: 'InterOpt: Improved gene expression quantification in qPCR experiments
      using weighted aggregation of reference genes'
  source_content_hash: sha256:a75af858023939ec3458e029b695fbe5a2939cadbbeaceee71a38356cae83448
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/asalimih/InterOpt
  tools:
  - InterOpt
  - R
  - devtools
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# subsample-stability-benchmarking

## Summary

Benchmarks reference-gene weighting methods (geom_sd_hybrid/geom(sd+), geom_sd, geom) across decreasing sample sizes by repeatedly subsampling a qPCR array dataset and comparing the mean SD stability of weighted RG2 combinations, demonstrating the small-sample superiority of geom(sd+).

## When to use

Use when evaluating how a reference-gene weighting method's stability performance degrades at low qPCR sample sizes, e.g. reproducing a Figure 2C-style mean-SD-versus-n comparison via repeated random subsampling.

## When NOT to use

- The dataset is a normalized high-throughput expression matrix rather than raw CT values from a qPCR array, since weights and stability measures here are computed on CT values with ctVal_source = TRUE
- The goal is selecting the best RG combination for a single full-size experiment rather than characterizing performance as a function of sample size
- Aggregating more than 5 reference genes is intended, since geom(sd) does not have expected benefits over the regular geometric mean in that regime

## Inputs

- Packaged raw CT matrix (e.g. data_GSE50013) with miRNAs as rows and samples as columns
- Corresponding sample groups vector (e.g. groups_GSE50013)
- Set of decreasing target sample sizes for subsampling
- Weighting method names: 'geom_sd_hybrid', 'geom_sd', 'geom'
- Number of subsampling repetitions (20) and combination size k = 2

## Outputs

- Per-repetition SD stability measures for all RG2 combinations per weighting method
- Aggregated table of mean SD versus sample size for each weighting method (Figure 2C-style analysis)
- CSV file of the sample-size stability comparison (e.g. sample_size_stability_results.csv)

## How to apply

Load the InterOpt R package (installed via devtools::install_github('asalimih/InterOpt')) together with a packaged CT matrix (e.g. data_GSE50013) and its sample groups. Restrict the combination search space to miRNAs with CT values below 37 in all samples using rowSums(data > 37) == 0. For each decreasing sample size, draw 20 random subsamples and call run_experiment with ctVal_source = TRUE, k = 2, norm_method = 'high_exp' with norm_method_exp_thr = 35, and weight_methods covering 'geom_sd_hybrid', 'geom_sd', and 'geom', parallelizing with mc.cores. For each repetition, extract the SD stability measure of all weighted RG2 combinations per weighting method and compute the mean SD across combinations, then aggregate the 20 repetitions into a mean-SD-versus-sample-size table. The rationale is that geom(sd+) (geom_sd_hybrid) is the improved variant for small sample sizes, so its advantage over geom(sd) should widen as n decreases; verify geom_sd_hybrid (and geom_sd) yields lower mean SD than plain geom at the smallest sample sizes before writing the comparison to CSV.

## Related tools

- **InterOpt** (R package providing run_experiment and calcWeight to compute weighted RG2 combinations, their aggregation weights, and SD/CV stability measures on subsampled CT data) — https://github.com/asalimih/InterOpt
- **devtools** (Used to install the InterOpt package from GitHub before running the benchmark)
- **R** (Execution environment for the subsampling loop, mc.cores parallelism, and aggregation of mean SD results)

## Examples

```
library(InterOpt); mirs = rownames(data_GSE50013)[rowSums(data_GSE50013>37)==0]; result = run_experiment(data_source=data_GSE50013, gr_source=groups_GSE50013, ctVal_source=T, sub_names=mirs, norm_method='high_exp', norm_method_exp_thr=35, k=2, weight_methods=c('geom','geom_sd','geom_sd_hybrid'), mc.cores=4)
```

## Evaluation signals

- geom_sd_hybrid (and geom_sd) yields lower mean SD than plain geom at the smallest sample sizes, with the gap over geom widening as sample size decreases
- Each sample size is covered by exactly 20 independent random subsamples before aggregation
- Only miRNAs with CT values below 37 in all samples enter the combination search space (rowSums(data > 37) == 0)
- run_experiment is configured with k = 2, norm_method = 'high_exp', norm_method_exp_thr = 35, ctVal_source = TRUE
- Output CSV contains a mean-SD-versus-sample-size table with one row per sample size for each of the three weighting methods

## Limitations

- Aggregating more than 5 RGs by the geom(sd) method does not have the expected benefits compared to the regular geometric mean
- NormFinder and Genorm stability measures in run_experiment require building the InterOptCuda CUDA kernel and adding it to PATH
- RG stability measure depends on gene expression distribution assumptions; other distributions (e.g. beta) and better measures are suggested for further improvement

## Evidence
