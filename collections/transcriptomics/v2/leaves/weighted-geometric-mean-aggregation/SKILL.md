---
name: weighted-geometric-mean-aggregation
description: Use when qPCR CT data contain two or more internal controls (e.g. RNU48,
  hsa-miR-16-5p) that must be combined into one normalization reference, or when the
  unweighted geometric mean of reference genes fails to yield stable target normalization
  (e.g. non-significant ΔΔCT results).
when_to_use_negative:
- 'Aggregating more than 5-6 reference genes: geom(sd) shows no expected benefit over
  the regular geometric mean beyond k≈6, and raw-CT-optimized weights over-fit beyond
  that point'
- Weight optimization from an external dataset (e.g. TCGA BRCA miRNA-seq) when a sufficiently
  large raw CT sample set (≥30 samples) is available — raw CT optimization is preferable
- Candidate genes with CT values ≥ 37 in any sample, which should be excluded before
  combination search
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3519
- http://edamontology.org/topic_0203
tools:
- name: InterOpt
  role: R package providing calcWeight (closed-form geom_sd_plus weight optimization
    from raw CT values) and run_experiment (combination search with stability measures
    SD/CV and aggregation of reference genes)
  repo: https://github.com/asalimih/InterOpt
- name: InterOptCuda
  role: CUDA kernel used to compute GeNorm and NormFinder stability measures in parallel
    for many weighted combinations; built with nvcc and placed in PATH
  repo: https://github.com/asalimih/InterOptCuda
- name: devtools
  role: Used to install the InterOpt development version from GitHub
- name: NormiRazor
  role: Source of the modified CUDA kernel underlying InterOptCuda's GeNorm/NormFinder
    computation
provenance:
  source_task_ids:
  - task_005
  - task_006
  - task_001
  - task_002
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2023.107945
    title: 'InterOpt: Improved gene expression quantification in qPCR experiments
      using weighted aggregation of reference genes'
  source_content_hash: sha256:135c9ad5885d71f50b3c99d1ad45923675db98d293f3b6d1ef6d6f409eb3e5b6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/asalimih/InterOpt
  tools:
  - InterOpt
  - InterOptCuda
  - NormiRazor
  - R
  - devtools
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3519
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 15
---

# weighted-geometric-mean-aggregation

## Summary

Aggregates multiple qPCR reference genes (internal controls) into a single 'virtual reference gene' using an optimally weighted geometric mean, with weights computed by InterOpt's calcWeight to minimize the SD (or CV) of the aggregated log-expression. Replaces the conventional unweighted geometric mean used in ΔΔCT normalization.

## When to use

Use when qPCR CT data contain two or more internal controls (e.g. RNU48, hsa-miR-16-5p) that must be combined into one normalization reference, or when the unweighted geometric mean of reference genes fails to yield stable target normalization (e.g. non-significant ΔΔCT results).

## When NOT to use

- Aggregating more than 5-6 reference genes: geom(sd) shows no expected benefit over the regular geometric mean beyond k≈6, and raw-CT-optimized weights over-fit beyond that point
- Weight optimization from an external dataset (e.g. TCGA BRCA miRNA-seq) when a sufficiently large raw CT sample set (≥30 samples) is available — raw CT optimization is preferable
- Candidate genes with CT values ≥ 37 in any sample, which should be excluded before combination search

## Inputs

- Raw CT value matrix (rows = internal controls, columns = samples), e.g. data_GSE78870 subset to controls like RNU48 and hsa-miR-16-5p
- Sample group labels for grouped experiments (e.g. groups_GSE78870, groups_GSE50013)
- Candidate reference gene list filtered to CT < 37 in all samples
- Optionally, a normalized external high-throughput dataset (e.g. TCGA BRCA miRNA-seq, data_TCGA_BRCA) for weight optimization when few qPCR samples exist

## Outputs

- Per-gene optimal weight vector w (real-valued, sums to 1) for the chosen weight_method
- Virtual reference gene values per sample (colSums(w * x) on raw CT, or result$aggregated_refs on normalized data)
- Combination table with Gene1/Gene2, w1, w2, CV, and SD per pair (result$res_source$geom_sd_hybrid)
- Normalized target gene values (ΔCT = target CT minus virtual reference CT)

## How to apply

Assemble a matrix x of raw CT values (one row per internal control, columns are samples) and compute optimal weights with calcWeight(x, ctVal = TRUE, weight_method = 'geom_sd_plus'), which yields the closed-form solution (Equations 15-19, covariance-based Equation 25) minimizing the SD of the weighted geometric mean's logarithm. Form the virtual reference gene as the weighted mean via colSums(w * x), then normalize targets by subtracting the virtual reference CT from target CT (ΔΔCT-compatible). For benchmarking many candidate combinations, use run_experiment with k=2 (or larger k), norm_method='high_exp', norm_method_exp_thr=35, and weight_methods=c('geom','geom_sd_hybrid'), restricting candidates to miRNAs with CT < 37 in all samples (rowSums(data>37)==0). The geom(sd) scheme (especially with weights from raw CT values) outperformed geom(cv), arith(sd), arith(cv), and the unweighted geometric mean in aggregated stability; validate that the weighted combination attains lower SD than the unweighted geometric mean and that weights are real-valued and sum to 1.

## Related tools

- **InterOpt** (R package providing calcWeight (closed-form geom_sd_plus weight optimization from raw CT values) and run_experiment (combination search with stability measures SD/CV and aggregation of reference genes)) — https://github.com/asalimih/InterOpt
- **InterOptCuda** (CUDA kernel used to compute GeNorm and NormFinder stability measures in parallel for many weighted combinations; built with nvcc and placed in PATH) — https://github.com/asalimih/InterOptCuda
- **devtools** (Used to install the InterOpt development version from GitHub)
- **NormiRazor** (Source of the modified CUDA kernel underlying InterOptCuda's GeNorm/NormFinder computation)

## Examples

```
library(InterOpt); w = calcWeight(x, ctVal = TRUE, weight_method = 'geom_sd_plus'); new_control = colSums(w*x)
```

## Evaluation signals

- Each weight vector is real-valued and sums to 1 (w1 + w2 ≈ 1 in the combination tables)
- The geom(sd)-weighted virtual reference gene attains a lower cross-sample SD (and CV) than the corresponding unweighted geometric mean aggregation
- In the 12-pair breast cancer validation, miR-21-5p ΔΔCT normalized by the geom(sd)-weighted mean of miR-16-5p/miR-361-5p/RNU48 yields a significant paired t-test where the unweighted geometric mean does not
- run_experiment output res_source$geom_sd_hybrid contains columns Gene1, Gene2, w1, w2, CV, SD, and aggregated_refs contains per-sample weighted mean CT values
- When reproducing the weight-source comparison, the raw-CT-optimized virtual RG attains SD at or below that of TCGA-derived weights at 30+ qPCR samples

## Limitations

- geom(sd) aggregation is beneficial only up to about 5-6 reference genes; with weights from raw CT values an over-fitting pattern appears beyond that
- The experimental validation of hsa-miR-21-5p used a low number of samples (12 pairs)
- RG stability measure depends on the gene expression distribution assumption; other distributions (e.g. beta distribution) and better measures are suggested for further improvement
- GeNorm/NormFinder measures require building and installing the InterOptCuda executable with nvcc and adding it to the PATH

## Evidence
