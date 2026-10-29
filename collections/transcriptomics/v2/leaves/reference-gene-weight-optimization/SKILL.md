---
name: reference-gene-weight-optimization
description: Use when raw CT values (or a compatible normalized high-throughput dataset)
  are available for 2 or more internal controls and a weighted mean must replace the
  unweighted geometric mean for ΔΔCT normalization.
when_to_use_negative:
- When aggregating more than ~5 reference genes — geom(sd) does not have expected
  benefits over the regular geometric mean in that regime
- When only a very small qPCR sample set (e.g. ~20 samples) is available and no biologically
  compatible external dataset exists for weight optimization
- When a single reference gene suffices or the experiment design does not permit computing
  CT values for multiple internal controls
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3519
- http://edamontology.org/topic_0204
tools:
- name: InterOpt
  role: R package providing calcWeight for optimal weight computation and run_experiment
    for screening weighted reference-gene combinations
  repo: https://github.com/asalimih/InterOpt
- name: devtools
  role: Used to install the InterOpt package from GitHub via install_github
- name: InterOptCuda
  role: GPU (CUDA) kernel used to compute NormFinder and Genorm stability measures
    in parallel for many combinations; must be built with nvcc and added to PATH
  repo: https://github.com/asalimih/InterOptCuda
- name: R
  role: Runtime environment in which the InterOpt weighting and aggregation pipeline
    is executed
provenance:
  source_task_ids:
  - task_002
  - task_005
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2023.107945
    title: 'InterOpt: Improved gene expression quantification in qPCR experiments
      using weighted aggregation of reference genes'
  source_content_hash: sha256:174fd24740476cae338f8f15e9c955a4eb46a80b651f1ade8d1a385d44c84163
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
  - http://edamontology.org/topic_3519
  - http://edamontology.org/topic_0204
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# reference-gene-weight-optimization

## Summary

Computes optimal aggregation weights for combining multiple qPCR reference genes (internal controls) into a single virtual reference gene via weighted mean, using the InterOpt weighting methods (geom(sd), geom(cv), arith(sd), arith(cv)), with geom(sd) as the best-performing candidate.

## When to use

Use when raw CT values (or a compatible normalized high-throughput dataset) are available for 2 or more internal controls and a weighted mean must replace the unweighted geometric mean for ΔΔCT normalization.

## When NOT to use

- When aggregating more than ~5 reference genes — geom(sd) does not have expected benefits over the regular geometric mean in that regime
- When only a very small qPCR sample set (e.g. ~20 samples) is available and no biologically compatible external dataset exists for weight optimization
- When a single reference gene suffices or the experiment design does not permit computing CT values for multiple internal controls

## Inputs

- Raw CT value matrix (rows = internal control genes, columns = samples), e.g. data_GSE78870
- Sample group annotations (e.g. groups_GSE78870)
- Optional external normalized high-throughput dataset (e.g. data_TCGA_BRCA) for weight optimization when few qPCR samples exist
- Candidate reference-gene list (optionally filtered to genes with CT < 37 in all samples)

## Outputs

- Per-gene optimal weight vectors for each weighting method (real-valued, summing to 1)
- Virtual reference gene expression profiles (weighted mean per sample)
- Stability measures (SD and CV) of each weighted combination, e.g. a table with Gene1, Gene2, w1, w2, CV, SD columns

## How to apply

Load the raw CT matrix (e.g. data_GSE78870 from the InterOpt package), subset rows to the chosen internal controls (e.g. RNU48, hsa-miR-16-5p), and compute weights with calcWeight(x, ctVal = TRUE, weight_method = 'geom_sd_plus'), which yields the closed-form/regression solution minimizing the SD of the log expression of the virtual reference gene. Aggregate the controls as the weighted mean (colSums(w * x)) and use it like a new internal control in the usual ΔΔCT workflow. For combination screening across many genes, call run_experiment with ctVal_source = TRUE, norm_method = 'high_exp', norm_method_exp_thr = 35, and weight_methods such as c('geom', 'geom_sd_hybrid'), restricting candidates to miRNAs with CT < 37 in all samples. Weights can alternatively be optimized on a biologically compatible external dataset (e.g. TCGA BRCA miRNA-seq with ctVal = FALSE), but raw-CT optimization is preferable when ≥30 qPCR samples are available. Validate that each weight vector is real-valued and sums to 1 and that the weighted combination attains lower SD/CV than the unweighted mean.

## Related tools

- **InterOpt** (R package providing calcWeight for optimal weight computation and run_experiment for screening weighted reference-gene combinations) — https://github.com/asalimih/InterOpt
- **devtools** (Used to install the InterOpt package from GitHub via install_github)
- **InterOptCuda** (GPU (CUDA) kernel used to compute NormFinder and Genorm stability measures in parallel for many combinations; must be built with nvcc and added to PATH) — https://github.com/asalimih/InterOptCuda
- **R** (Runtime environment in which the InterOpt weighting and aggregation pipeline is executed)

## Examples

```
library(InterOpt); controls = c('RNU48','hsa-miR-16-5p'); x = data_GSE78870[controls,]; w = calcWeight(x, ctVal = TRUE, weight_method = 'geom_sd_plus'); new_control = colSums(w*x)
```

## Evaluation signals

- Each computed weight vector is real-valued and sums to 1
- The geom(sd)-weighted virtual reference gene attains lower cross-sample SD than the corresponding unweighted geometric mean
- The output combination table contains expected columns (Gene1, Gene2, w1, w2, CV, SD) with SD values in a plausible CT range (e.g. ~1.2–2.3 for the GSE50013 example)
- In differential-expression validation, target-gene significance under geom(sd)-weighted normalization matches the reported outcome (e.g. miR-21-5p significant with weighted, not unweighted, normalization)
- For combination screening, candidate miRNAs are restricted to those with CT values less than 37 in all samples before running run_experiment

## Limitations

- Aggregating more than 5 RGs by the geom(sd) method does not have expected benefits compared to the regular geometric mean
- RG stability measure depends on the gene expression distribution assumption; other distributions (like beta) and better measures are suggested for further improvement
- Experimental validation of hsa-miR-21-5p used a low number of samples (12 pairs)
- Tries calculating weights from TCGA-derived normalized data performs worse than raw-CT optimization when only ~20 qPCR samples are available, but is on par at 30 samples

## Evidence
