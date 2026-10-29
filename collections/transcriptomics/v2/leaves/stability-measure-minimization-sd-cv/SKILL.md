---
name: stability-measure-minimization-sd-cv
description: Use when you have a raw CT matrix (rows = internal controls, columns
  = samples) for a chosen reference-gene combination and need per-gene weights that
  minimize the SD or CV of the weighted mean instead of taking a simple unweighted
  average.
when_to_use_negative:
- 'Aggregating more than 5 reference genes with geom(sd): the paper notes it ''does
  not have the expected benefits compared to the regular geometric mean'' at that
  scale'
- The goal is to select which genes to combine rather than to weight an already-chosen
  combination — use run_experiment-based combination scanning instead
- Data are already normalized high-throughput expression values when raw CT values
  are available for the geom(sd) method, since raw-CT-derived weights avoid gene-independent
  technical variation
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3519
- http://edamontology.org/topic_0203
tools:
- name: InterOpt
  role: R package providing calcWeight and run_experiment to compute the optimal SD/CV-minimizing
    aggregation weights from CT matrices
  repo: https://github.com/asalimih/InterOpt
- name: InterOptCuda
  role: CUDA kernel used by InterOpt to compute NormFinder and GeNorm stability measures
    in parallel for many combinations; optional for this weight-optimization skill
  repo: https://github.com/asalimih/InterOptCuda
- name: R
  role: Runtime environment in which the InterOpt package is installed and executed
    (via devtools::install_github)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2023.107945
    title: 'InterOpt: Improved gene expression quantification in qPCR experiments
      using weighted aggregation of reference genes'
  source_content_hash: sha256:59d7b749d1dda941dcf7053c5d71fb734e9162ecad27811a48503e1b302a720e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/asalimih/InterOpt
  tools:
  - InterOpt
  - InterOptCuda
  - R
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3519
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# stability-measure-minimization-sd-cv

## Summary

Computes optimal aggregation weights for combining multiple qPCR reference genes into a single virtual reference gene by minimizing the SD or CV of the weighted geometric or arithmetic mean, using the four InterOpt weighting methods (geom(sd), geom(cv), arith(sd), arith(cv)).

## When to use

Use when you have a raw CT matrix (rows = internal controls, columns = samples) for a chosen reference-gene combination and need per-gene weights that minimize the SD or CV of the weighted mean instead of taking a simple unweighted average.

## When NOT to use

- Aggregating more than 5 reference genes with geom(sd): the paper notes it 'does not have the expected benefits compared to the regular geometric mean' at that scale
- The goal is to select which genes to combine rather than to weight an already-chosen combination — use run_experiment-based combination scanning instead
- Data are already normalized high-throughput expression values when raw CT values are available for the geom(sd) method, since raw-CT-derived weights avoid gene-independent technical variation

## Inputs

- Matrix of raw CT values (x), one row per internal control, one column per sample (e.g., data_GSE78870)
- Sample group definitions (e.g., groups_GSE78870)
- List of selected reference-gene names forming the combination (e.g., c('RNU48','hsa-miR-16-5p'))

## Outputs

- Per-method weight vectors (geom(sd), geom(cv), arith(sd), arith(cv)) with one weight per gene, real-valued and summing to 1
- optimized_weights.csv table with columns: method, gene names, per-gene weights
- Weighted mean virtual reference-gene series usable as a new internal control in the ΔΔCT method

## How to apply

Load the raw CT matrix (e.g., data_GSE78870) and subset rows for the selected reference-gene combination. Compute geom(sd) weights via the closed-form/linear-regression solution using calcWeight(x, ctVal = TRUE, weight_method = 'geom_sd_plus'), which derives weights directly from raw CT values (Equations 15-19, covariance-based Equation 25) and is robust to gene-independent technical variation. For the other methods, solve the arith(cv) problem via the pseudo-inverse/eigenvector solution, geom(cv) via the stabilized Barzilai-Borwein gradient method iterated to CV convergence, and arith(sd) by exhaustive search over the weight simplex. Assemble the four real-valued weight vectors (each summing to 1) into a table with method, gene names, and per-gene weights, and write optimized_weights.csv; prefer geom(sd), which the paper shows outperforms the other methods in aggregated stability.

## Related tools

- **InterOpt** (R package providing calcWeight and run_experiment to compute the optimal SD/CV-minimizing aggregation weights from CT matrices) — https://github.com/asalimih/InterOpt
- **InterOptCuda** (CUDA kernel used by InterOpt to compute NormFinder and GeNorm stability measures in parallel for many combinations; optional for this weight-optimization skill) — https://github.com/asalimih/InterOptCuda
- **R** (Runtime environment in which the InterOpt package is installed and executed (via devtools::install_github))

## Examples

```
library(InterOpt); controls = c('RNU48','hsa-miR-16-5p'); x = data_GSE78870[controls,]; w = calcWeight(x, ctVal = TRUE, weight_method = 'geom_sd_plus')
```

## Evaluation signals

- Each computed weight vector is real-valued (negative weights are permitted, e.g., w1 = -0.01028321 in the README example) and sums to 1
- The geom(sd)-weighted combination attains a lower SD than the corresponding unweighted geometric mean, consistent with geom(sd)'s reported superiority in aggregated stability
- geom(sd) weights can be recomputed solely from raw CT values and remain unchanged by gene-independent technical variation
- Aggregated stability (SD/CV) of the weighted virtual reference gene across samples is at or below that of candidate unweighted combinations of the same genes
- Weighted mean output (e.g., colSums(w*x) or result$aggregated_refs) has one value per sample and falls in a plausible CT range for the input data

## Limitations

- Aggregating more than 5 RGs by geom(sd) does not have the expected benefits compared to the regular geometric mean
- RG stability measure depends on gene expression distribution assumptions; the paper suggests examining other distributions (like beta distribution) and better measures for further improvement
- NormFinder and GeGenorm stability measures via InterOpt require building the InterOptCuda CUDA kernel with nvcc and placing the executable on PATH

## Evidence
