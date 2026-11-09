---
name: roc-and-correlation-benchmarking
description: Use when you have a continuous per-cell score (e.g. signalling entropy
  rate) and binary or continuous ground-truth labels (pluripotent vs progenitor, or
  a pluripotency score) and need to quantify discriminative performance.
when_to_use_negative:
- The input is already a feature/engineered table and no per-cell score needs external
  validation — benchmarking is redundant.
- Cells lack ground-truth labels or pluripotency annotations, making ROC and correlation
  against truth impossible.
- The goal is pseudo-temporal ordering of cells within a lineage rather than benchmarking
  discriminative accuracy of a potency score.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SCENT
  role: Computes the normalized signalling entropy rate (SR) per cell that is then
    benchmarked by ROC AUC and Spearman correlation.
  repo: https://github.com/aet21/SCENT
- name: R
  role: Implementation environment in which SCENT is run and the per-cell SR values,
    ROC AUC, Spearman correlation, and CSV export are produced.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/ncomms15599
    title: Single-cell entropy for accurate estimation of differentiation potency
      from a cell’s transcriptome
  source_content_hash: sha256:0886633bc8d17b929463f96455b644bbe79ac8b7c87358711a1ab4c1ad464bfb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aet21/SCENT
  tools:
  - R
  - SCENT
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# roc-and-correlation-benchmarking

## Summary

Benchmark a per-cell potency score such as signalling entropy (SCENT SR) against ground-truth class labels by computing Spearman correlation with a pluripotency score and ROC AUC for pluripotent-vs-non-pluripotent discrimination.

## When to use

Use when you have a continuous per-cell score (e.g. signalling entropy rate) and binary or continuous ground-truth labels (pluripotent vs progenitor, or a pluripotency score) and need to quantify discriminative performance.

## When NOT to use

- The input is already a feature/engineered table and no per-cell score needs external validation — benchmarking is redundant.
- Cells lack ground-truth labels or pluripotency annotations, making ROC and correlation against truth impossible.
- The goal is pseudo-temporal ordering of cells within a lineage rather than benchmarking discriminative accuracy of a potency score.

## Inputs

- Per-cell signalling entropy rate (SR) vector from SCENT
- Binary class labels (pluripotent hESC vs non-pluripotent progenitor) for each cell
- Continuous pluripotency score per cell
- Single-cell RNA-Seq expression matrix (e.g. Chu et al., 1,018 cells)
- Human PPI (connectome) network

## Outputs

- Spearman correlation coefficient between SR and pluripotency score
- ROC curve and AUC value for pluripotency discrimination
- Wilcoxon rank-sum test P-value comparing SR across classes
- CSV file of per-cell SR values with class labels (chu_scent_sr.csv)

## How to apply

Compute SCENT SR per cell in R by integrating the single-cell RNA-Seq expression matrix (e.g. the 1,018-cell Chu et al. hESC/progenitor set) with a human PPI connectome network, without feature selection or prior marker knowledge. Correlate the per-cell SR values with the pluripotency score using Spearman correlation (target: rho = 0.91 in the Chu et al. replication). Then build an ROC curve classifying pluripotent hESCs versus non-pluripotent progenitors from the SR values and compute the AUC (target: AUC = 0.96). Validate group separation with a Wilcoxon rank-sum test (expected P < 1e-300). Persist per-cell SR values and class labels to a CSV file (e.g. chu_scent_sr.csv) so the ROC and correlation analyses are reproducible.

## Related tools

- **SCENT** (Computes the normalized signalling entropy rate (SR) per cell that is then benchmarked by ROC AUC and Spearman correlation.) — https://github.com/aet21/SCENT
- **R** (Implementation environment in which SCENT is run and the per-cell SR values, ROC AUC, Spearman correlation, and CSV export are produced.)

## Examples

```
devtools::install_github("aet21/SCENT")
```

## Evaluation signals

- Spearman correlation between per-cell SR and pluripotency score equals approximately 0.91.
- ROC AUC for discriminating pluripotent hESCs from non-pluripotent progenitors equals approximately 0.96.
- Wilcoxon rank-sum test on SR between pluripotent and non-pluripotent classes yields P < 1e-300.
- Per-cell SR values and class labels are written to chu_scent_sr.csv with one row per cell (1,018 cells for Chu et al.).
- SR is computed without feature selection or prior biological/marker knowledge, matching the SCENT design.

## Limitations

- High dropout rates in scRNA-Seq may preclude marker-based scoring, which is exactly the setting SCENT targets; but benchmarking still requires trusted labels.
- SCENT's comparative advantage is over transcriptomic entropy measures without network information (StemID, SLICE) and pseudo-time tools (Monocle, MPath); the benchmark does not validate potency ordering within a single lineage.
- No existing assay can measure all single cells in a population, limiting independent validation of potency estimates.

## Evidence

- [results] Signalling entropy provided a highly accurate discriminator of pluripotency versus non-pluripotency at the single-cell level in the Chu et al. data, achieving an AUC of 0.96 (Wilcoxon test P<1e-300).: [evidence span withheld — anchor status: not_found]
- [discussion] signalling entropy can accurately discriminate pluripotent from multipotent and differentiated cells, without the need for feature selection or training: "signalling entropy can accurately discriminate pluripotent from multipotent and differentiated cells, without the need for feature selection or training"
- [readme] The main purpose of the `SCENT` package is to provide a means of estimating the differentiation potency of single cells without the need to assume prior biological knowledge such as marker expression or timepoint.: "estimating the differentiation potency of single cells without the need to assume prior biological knowledge such as marker expression or timepoint"
- [abstract] Signalling entropy provides a more accurate and robust potency estimate than other entropy-based measures: [evidence span withheld — anchor status: not_found]
