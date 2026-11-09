---
name: cross-dataset-weight-comparison
description: Decision support for choosing between raw-CT optimization and external-dataset
  (e.g. TCGA) weight optimization when only ~20–30 qPCR samples and a few internal
  controls are available.
when_to_use_negative:
- When a sufficiently large qPCR cohort is available, raw CT optimization on the full
  sample set is preferable — external-dataset weights were only on par at 30 samples
- When the external dataset is not biologically compatible with the qPCR experiment
  (the paper explicitly required a compatible external dataset, e.g. matched tissue
  type/cancer)
- When aggregating more than 5 reference genes — geom(sd) weighting 'does not have
  the expected benefits compared to the regular geometric mean'
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0204
tools:
- name: InterOpt
  role: R package providing calcWeight and bundled datasets (data_GSE78870, data_TCGA_BRCA)
    used to compute geom_sd_plus/geom_sd weights and build virtual reference genes
  repo: https://github.com/asalimih/InterOpt
- name: devtools
  role: Used to install the InterOpt development version from GitHub
  repo: https://github.com/asalimih/InterOpt
- name: R
  role: Analysis environment in which the weight computation and comparison table
    are generated
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2023.107945
    title: 'InterOpt: Improved gene expression quantification in qPCR experiments
      using weighted aggregation of reference genes'
  source_content_hash: sha256:f52596947797c3c33f8f8006be4d6bc59f176bf0d58d269c3a28735f58ecade8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/asalimih/InterOpt
  tools:
  - InterOpt
  - R
  - devtools
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0204
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# cross-dataset-weight-comparison

## Summary

Compares virtual reference gene aggregation weights (geom(sd)/geom(sd+)) computed from raw CT values of a small qPCR sample subset against weights computed from a normalized external high-throughput dataset (TCGA BRCA miRNA-seq), assembling weights and stability measures (SD, CV) into one comparison table.

## When to use

Decision support for choosing between raw-CT optimization and external-dataset (e.g. TCGA) weight optimization when only ~20–30 qPCR samples and a few internal controls are available.

## When NOT to use

- When a sufficiently large qPCR cohort is available, raw CT optimization on the full sample set is preferable — external-dataset weights were only on par at 30 samples
- When the external dataset is not biologically compatible with the qPCR experiment (the paper explicitly required a compatible external dataset, e.g. matched tissue type/cancer)
- When aggregating more than 5 reference genes — geom(sd) weighting 'does not have the expected benefits compared to the regular geometric mean'

## Inputs

- Raw CT matrix from data_GSE78870 (rows = internal controls, columns = samples), subset to controls such as RNU48 and hsa-miR-16-5p over a 20–30 sample subset
- Normalized external dataset data_TCGA_BRCA (TCGA BRCA miRNA-seq) with groups_TCGA_BRCA
- InterOpt package installed via devtools::install_github('asalimih/InterOpt')

## Outputs

- weight_source_comparison.csv containing per-control weights (w1/w2), CV, and SD for both the raw-CT-optimized and TCGA-derived weight sets
- Two candidate virtual reference genes (weighted means colSums(w * x)) with cross-sample SD and CV stability measures

## How to apply

Load the GSE78870 raw CT matrix from the InterOpt package, subset it to the internal controls (e.g. RNU48 and hsa-miR-16-5p) over a 30-sample subset, and compute geom_sd_plus weights directly from raw CT with calcWeight(x, ctVal = TRUE, weight_method = 'geom_sd_plus'). Independently, normalize the TCGA BRCA miRNA expression dataset (data_TCGA_BRCA, ctVal = FALSE) and recompute geom_sd weights on the normalized values as the external optimization source. Build the two candidate virtual reference genes as weighted means (colSums(w * x)) of the controls under each weight set, then compute cross-sample SD and CV for each. Assemble weights (w1/w2), CV, and SD into a single table (weight_source_comparison.csv) and prefer raw CT optimization when enough samples exist — the paper found TCGA-derived weights better at 20 samples but only on par at 30 samples.

## Related tools

- **InterOpt** (R package providing calcWeight and bundled datasets (data_GSE78870, data_TCGA_BRCA) used to compute geom_sd_plus/geom_sd weights and build virtual reference genes) — https://github.com/asalimih/InterOpt
- **devtools** (Used to install the InterOpt development version from GitHub) — https://github.com/asalimih/InterOpt
- **R** (Analysis environment in which the weight computation and comparison table are generated)

## Examples

```
library(InterOpt); controls = c('RNU48','hsa-miR-16-5p'); x = data_GSE78870[controls,]; w = calcWeight(x, ctVal = TRUE, weight_method = 'geom_sd_plus'); new_control = colSums(w*x)
```

## Evaluation signals

- The output table contains w1/w2, CV, and SD columns for both optimization sources (raw CT and TCGA-derived)
- The raw-CT-optimized virtual reference gene attains SD at or below the SD of the TCGA-derived weights when enough qPCR samples are used, reproducing the paper's preference for raw CT optimization
- Weights sum to 1 per combination (e.g. w1 + w2 = 1, as in the README's geom_sd_hybrid output where 0.12348869 + 0.8765113 ≈ 1)
- The internal controls (RNU48, hsa-miR-16-5p) match those preselected in the experiment, and the subset size (20 or 30 samples) matches the evaluated sample sizes
- The TCGA side of the comparison uses ctVal = FALSE (normalized values) while the qPCR side uses ctVal = TRUE (raw CT)

## Limitations

- External-dataset-derived weights only outperformed raw-CT weights at very low sample sizes (20 qPCR samples); at 30 samples they were on par, so raw CT optimization is preferable when enough samples exist
- RG stability measures depend on the assumed gene expression distribution; the paper suggests examining other distributions (like beta distribution) for further improvement
- Experimental validation of hsa-miR-21-5p used a low number of samples (12 pairs)

## Evidence

- [readme] README: raw CT weight computation: [evidence span withheld — anchor status: not_found]
- [readme] README: weighted mean construction: "new_control = colSums(w*x)"
- [results] Article: sample size evaluation with TCGA: "the qPCR array dataset is evaluated in three different sample sizes, 20, 30, and 106"
- [results] Article: TCGA as external dataset: "The Cancer Genome Atlas (TCGA) breast cancer miRNA expression dataset was obtained as a biologically compatible external dataset"
- [discussion] Article: weight source decision: "the weights would be either calculated based on the raw CT values or the normalized high-throughput dataset"
- [results] Article: geom(sd) best on raw CT: "on both datasets the geom(sd) method outperforms all other methods in terms of the aggregated stability, specifically when the aggregation weights are calculated based on the raw CT values"
