---
name: reference-gene-combination-benchmarking
description: Use when you have raw CT qPCR array data with many candidate reference
  genes/miRNAs and need to select and benchmark the most stable weighted combination
  (virtual reference gene) across candidate weighting schemes.
when_to_use_negative:
- You only have 2-3 preselected internal controls and no combinatorial search is needed
  — use calcWeight directly instead of run_experiment
- Aggregating more than 5 reference genes — geom(sd) does not have expected benefits
  compared to the regular geometric mean in that regime
- Input data are already normalized expression values without raw CT values and sample
  size does not support weight optimization on normalized data
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_3519
- http://edamontology.org/topic_0203
tools:
- name: InterOpt
  role: R package that runs the weighted reference-gene combination experiment (run_experiment,
    calcWeight) and computes SD/CV stability measures
  repo: https://github.com/asalimih/InterOpt
- name: InterOptCuda
  role: CUDA kernel computing GeNorm and NormFinder stability measures in parallel
    across many combinations; built with nvcc and added to PATH
  repo: https://github.com/asalimih/InterOptCuda
- name: devtools
  role: Used to install the InterOpt development version from GitHub
  repo: https://github.com/asalimih/InterOpt
- name: GeNorm
  role: Pairwise-variation stability measure computed over weighted combinations via
    InterOptCuda
- name: NormFinder
  role: Model-based stability measure computed over weighted combinations via InterOptCuda
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2023.107945
    title: 'InterOpt: Improved gene expression quantification in qPCR experiments
      using weighted aggregation of reference genes'
  source_content_hash: sha256:5258a3154e45cd87aa866a271012d1b413a9e16b58a9469b9bb6929545518e5e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/asalimih/InterOpt
  tools:
  - GeNorm
  - InterOpt
  - InterOptCuda
  - NormFinder
  - R
  - devtools
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_3519
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# reference-gene-combination-benchmarking

## Summary

Benchmarks weighted aggregation methods (geom(sd), geom(cv), arith(sd), arith(cv), plus the unweighted geometric mean) for combining qPCR reference genes into a stable virtual reference, using the InterOpt R package on qPCR array datasets with SD, CV, GeNorm, and NormFinder stability measures.

## When to use

Use when you have raw CT qPCR array data with many candidate reference genes/miRNAs and need to select and benchmark the most stable weighted combination (virtual reference gene) across candidate weighting schemes.

## When NOT to use

- You only have 2-3 preselected internal controls and no combinatorial search is needed — use calcWeight directly instead of run_experiment
- Aggregating more than 5 reference genes — geom(sd) does not have expected benefits compared to the regular geometric mean in that regime
- Input data are already normalized expression values without raw CT values and sample size does not support weight optimization on normalized data

## Inputs

- Raw CT qPCR array matrix (genes/miRNAs x samples), e.g. data_GSE50013
- Sample group vector, e.g. groups_GSE50013
- Optional external dataset for weight optimization, e.g. TCGA breast cancer miRNA expression dataset (data_target/gr_target)
- InterOpt R package with InterOptCuda executable in PATH for GeNorm/NormFinder measures

## Outputs

- Per-combination optimal aggregation weights (result$res_source$<weight_method>, columns Gene1, Gene2, w1, w2, CV, SD)
- Aggregated virtual reference values per sample (result$aggregated_refs$<weight_method>)
- Stability measures (SD, CV, GeNorm, NormFinder) per weighted combination
- Comparison table, e.g. stability_benchmark_results.csv

## How to apply

Load the packaged qPCR array matrices (e.g. data_GSE78870/groups_GSE78870 for breast cancer tissue, data_GSE50013/groups_GSE50013 for liver cancer plasma) from InterOpt and restrict candidates to genes with CT values less than 37 in all samples (rowSums(data>37)==0). Call run_experiment() with k=2 pair combinations, norm_method='high_exp' with norm_method_exp_thr=35, ctVal_source=TRUE so weights are optimized on raw CT values, output_agg_refs=TRUE, and mc.cores=4 for parallelization, covering weight_methods such as 'geom', 'geom_sd', 'geom_cv', 'arith_sd', 'arith_cv', and 'geom_sd_hybrid'. The data is automatically normalized by the chosen norm_method, and weights are computed on the normalized data unless ctVal_source=TRUE is set. To add GeNorm and NormFinder measures, build InterOptCuda with nvcc, put the executable in PATH, and add algors=c('SD','CV','Genorm','NormFinder') to run_experiment. Compare methods by assembling per-method weights (e.g. result$res_source$geom_sd_hybrid), aggregated reference values (result$aggregated_refs), and the stability measures into comparison tables; on both benchmark datasets geom(sd) outperformed all other methods, especially with weights from raw CT values.

## Related tools

- **InterOpt** (R package that runs the weighted reference-gene combination experiment (run_experiment, calcWeight) and computes SD/CV stability measures) — https://github.com/asalimih/InterOpt
- **InterOptCuda** (CUDA kernel computing GeNorm and NormFinder stability measures in parallel across many combinations; built with nvcc and added to PATH) — https://github.com/asalimih/InterOptCuda
- **devtools** (Used to install the InterOpt development version from GitHub) — https://github.com/asalimih/InterOpt
- **GeNorm** (Pairwise-variation stability measure computed over weighted combinations via InterOptCuda)
- **NormFinder** (Model-based stability measure computed over weighted combinations via InterOptCuda)

## Examples

```
result = run_experiment(data_source=data_GSE50013, gr_source=groups_GSE50013, ctVal_source=T, sub_names=rownames(data_GSE50013)[rowSums(data_GSE50013>37)==0], norm_method='high_exp', norm_method_exp_thr=35, k=2, weight_methods=c('geom','geom_sd_hybrid'), output_agg_refs=T, mc.cores=4)
```

## Evaluation signals

- Filtering invariant: sub_names equals rownames(data)[rowSums(data>37)==0], i.e. every retained miRNA has CT < 37 in all samples
- run_experiment returns res_source per weight_method with columns Gene1, Gene2, w1, w2, CV, SD; weights per combination sum to 1 (small negative weights allowed, e.g. -0.01028321)
- aggregated_refs contains the per-sample weighted mean for each combination and matches the reported w1/w2 weights
- GeNorm/NormFinder rows appear in output only when InterOptCuda is in PATH and algors=c('SD','CV','Genorm','NormFinder') was passed
- Benchmark expectation: geom(sd) should show the lowest aggregated stability across SD/CV/GeNorm/NormFinder, particularly with ctVal_source=TRUE

## Limitations

- Aggregating more than 5 reference genes by geom(sd) does not have expected benefits compared to the regular geometric mean
- RG stability measure depends on the assumed gene expression distribution; other distributions (like beta distribution) and better measures are suggested for future improvement
- GeNorm and NormFinder measures require building InterOptCuda with nvcc and adding the executable to PATH; on unsupported CUDA setups these measures are unavailable
- In each sample NormFinder only uses elements with CT<35 when calculating the sample mean in the CUDA kernel

## Evidence
