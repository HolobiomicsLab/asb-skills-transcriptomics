---
name: scale-invariant-aggregation-optimization
description: Use when raw CT values or normalized expression matrices for multiple
  candidate reference genes (internal controls) are available and a single optimally
  weighted virtual reference gene is needed for ΔΔCT normalization.
when_to_use_negative:
- Aggregating more than 5 reference genes with geom(sd) — the article reports no expected
  benefit compared to the regular geometric mean in that regime
- Stability measures other than SD/CV are required and the InterOptCuda executable
  is not installed in PATH (Genorm/NormFinder cannot be computed)
- The input is already a single aggregated reference value or a target-gene expression
  table with no multiple reference genes to weight
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_3512
tools:
- name: InterOpt
  role: R package providing calcWeight and run_experiment to compute optimal aggregation
    weights and stability measures (SD, CV) of weighted reference-gene combinations
  repo: https://github.com/asalimih/InterOpt
- name: InterOptCuda
  role: CUDA kernel used by InterOpt to calculate Genorm and NormFinder stability
    measures in parallel across many combinations; optional add-on built with nvcc
    and placed in PATH
  repo: https://github.com/asalimih/InterOptCuda
- name: devtools
  role: Used to install the InterOpt development version from GitHub via devtools::install_github
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2023.107945
    title: 'InterOpt: Improved gene expression quantification in qPCR experiments
      using weighted aggregation of reference genes'
  source_content_hash: sha256:2d9bf9a59ba83a62fcb85c29881960425171f24da2fa92af5daa4d70ff099c27
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/asalimih/InterOpt
  tools:
  - InterOpt
  - InterOptCuda
  - R
  - devtools
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4375
n_claims: 16
---

# scale-invariant-aggregation-optimization

## Summary

Computes optimal aggregation weights for combining multiple qPCR reference genes into a single virtual reference gene via weighted arithmetic or geometric means, minimizing SD or CV (the four InterOpt methods: geom(sd), geom(cv), arith(sd), arith(cv)).

## When to use

Use when raw CT values or normalized expression matrices for multiple candidate reference genes (internal controls) are available and a single optimally weighted virtual reference gene is needed for ΔΔCT normalization.

## When NOT to use

- Aggregating more than 5 reference genes with geom(sd) — the article reports no expected benefit compared to the regular geometric mean in that regime
- Stability measures other than SD/CV are required and the InterOptCuda executable is not installed in PATH (Genorm/NormFinder cannot be computed)
- The input is already a single aggregated reference value or a target-gene expression table with no multiple reference genes to weight

## Inputs

- Matrix of raw CT values (rows = internal control reference genes, columns = samples)
- Sample group annotations (e.g. groups_GSE78870)
- Candidate reference-gene combination (subset of gene row names)
- Choice of weighting method: geom(sd), geom(cv), arith(sd), or arith(cv)

## Outputs

- Per-gene optimal weight vector (real-valued, summing to 1) for each weighting method
- Table of weights and stability measures (CV, SD) per gene combination (e.g. result$res_source$geom_sd_hybrid)
- Aggregated virtual reference gene expression matrix (e.g. result$aggregated_refs)

## How to apply

Load a raw CT matrix (e.g. data_GSE78870) and subset rows of the selected reference-gene combination (one row per internal control). Compute geom(sd) weights with calcWeight(x, ctVal = TRUE, weight_method = 'geom_sd_plus'), which uses the closed-form/linear-regression solution (Equations 15-19, covariance-based Equation 25) directly on raw CT values. For arith(cv), solve the pseudo-inverse/eigenvector solution of the CV-minimization problem; for geom(cv), use the stabilized Barzilai-Borwein gradient solution iterated until the weighted geometric mean's CV converges; for arith(sd), perform exhaustive search over the weight simplex retaining the vector minimizing the SD of the weighted arithmetic mean. Aggregate per-gene weights into a table (method, gene names, per-gene weights), then validate that each weight vector is real-valued and sums to 1. When screening many gene pairs, use run_experiment with weight_methods such as 'geom' and 'geom_sd_hybrid' over candidate combinations (e.g. miRNAs with CT < 37 in all samples).

## Related tools

- **InterOpt** (R package providing calcWeight and run_experiment to compute optimal aggregation weights and stability measures (SD, CV) of weighted reference-gene combinations) — https://github.com/asalimih/InterOpt
- **InterOptCuda** (CUDA kernel used by InterOpt to calculate Genorm and NormFinder stability measures in parallel across many combinations; optional add-on built with nvcc and placed in PATH) — https://github.com/asalimih/InterOptCuda
- **devtools** (Used to install the InterOpt development version from GitHub via devtools::install_github)

## Examples

```
library(InterOpt); w = calcWeight(data_GSE78870[c('RNU48','hsa-miR-16-5p'),], ctVal = TRUE, weight_method = 'geom_sd_plus'); new_control = colSums(w*x)
```

## Evaluation signals

- Each computed weight vector must be real-valued and sum to 1
- The geom(sd)-weighted combination must attain a lower SD than the corresponding unweighted geometric mean, consistent with the reported superiority of geom(sd)
- Result table rows must contain Gene1, Gene2, per-gene weights (w1, w2), and CV/SD columns for the chosen weight_method
- geom(sd) should outperform the other weighting methods in aggregated stability, especially when weights are calculated from raw CT values

## Limitations

- Aggregating more than 5 RGs by geom(sd) does not have the expected benefits compared to the regular geometric mean
- RG stability measure depends on gene expression distribution assumptions; other distributions (like beta distribution) and better measures are suggested for future improvement
- Experimental validation of hsa-miR-21-5p used a low number of samples (12 pairs)

## Evidence
