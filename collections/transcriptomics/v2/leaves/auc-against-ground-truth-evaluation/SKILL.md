---
name: auc-against-ground-truth-evaluation
description: Use when a method's ability to detect differential regulation (or other
  binary labels) must be quantified on simulated data with known ground truth, and
  compared to baseline/alternative methods via AUC.
when_to_use_negative:
- Evaluating on real patient data where no ground-truth differential regulation labels
  exist (the article notes such findings need replication by additional patients and
  controls rather than AUC scoring)
- Assessing clustering, embedding, or subpopulation-matching quality — the article
  validates those steps separately using surrogate labels from bulk data, not AUC
  against simulated differential regulation
- The simulation design lacks a defined ground-truth differential-regulation structure,
  making AUC against 'known truth' undefined
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: scCompReg
  role: Provides the likelihood-ratio differential-regulation statistic (with p-value
    computation and FDR control) whose AUC is being evaluated on the simulated datasets
  repo: https://github.com/SUwonglab/sc-compReg
- name: GENIE3
  role: GRN inference method used to build the alternative differential-network pipelines
    (GENIE3 + diffK/diffRank) benchmarked against sc-compReg
- name: PIDC
  role: GRN inference method used to build the alternative differential-network pipelines
    (PIDC + diffK/diffRank) benchmarked against sc-compReg
- name: sc-compReg repository data/prior_data
  role: Source of prior_data and example data obtained via git lfs clone, used in
    constructing the simulation and evaluation runs
  repo: https://github.com/SUwonglab/sc-compReg
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-25089-2
    title: Sc-compReg enables the comparison of gene regulatory networks between conditions
      using single-cell data
  source_content_hash: sha256:03f2da76b71712073fa3a7d8254b78f7053c2de3fbbb2b2e05cb874e3c374beb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SUwonglab/sc-compReg
  tools:
  - GENIE3
  - PIDC
  - R (>=)
  - sc-compReg
  - sc-compReg repository data/prior_data
  - scCompReg
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# auc-against-ground-truth-evaluation

## Summary

Validates a differential regulatory detection method by computing AUC against known ground-truth differential regulation in simulated two-condition scRNA-seq/scATAC-seq data, and compares it head-to-head against baselines and alternative pipelines (e.g., sc-compReg LR at AUC 0.9157 vs scRNA-only baseline at 0.7561 in the combined-differential-regulation scenario).

## When to use

Use when a method's ability to detect differential regulation (or other binary labels) must be quantified on simulated data with known ground truth, and compared to baseline/alternative methods via AUC.

## When NOT to use

- Evaluating on real patient data where no ground-truth differential regulation labels exist (the article notes such findings need replication by additional patients and controls rather than AUC scoring)
- Assessing clustering, embedding, or subpopulation-matching quality — the article validates those steps separately using surrogate labels from bulk data, not AUC against simulated differential regulation
- The simulation design lacks a defined ground-truth differential-regulation structure, making AUC against 'known truth' undefined

## Inputs

- Simulated paired two-condition scRNA-seq and scATAC-seq samples with known ground-truth differential regulation labels
- Ground-truth differential-regulation label vector per scenario
- Prediction/statistic scores from each method (sc-compReg LR statistic, scRNA-only baseline scores, GENIE3/PIDC + diffK/diffRank differential scores)
- Consistent cluster assignments across simulated samples

## Outputs

- AUC value per scenario-method pair against ground truth
- AUC comparison table written to auc_results.csv
- Ranking of methods (e.g., sc-compReg 0.9157 vs baseline 0.7561 in the all-three-types scenario)

## How to apply

Simulate paired two-condition scRNA-seq and scATAC-seq datasets with known ground-truth differential regulatory structure under defined scenarios (differential TF expression only, differential RE accessibility only, differential TF-TG network structure only, and all three combined). Run the method under test (here the sc_compreg likelihood-ratio statistic with p-value computation and FDR control, using consistent cluster assignments) alongside the scRNA-only baseline and all alternative pipelines (GENIE3/PIDC crossed with diffK/diffRank, 8 alternatives total) on the same simulated datasets. Compute AUC against the known ground truth for every scenario-method pair (10 methods here: 1 sc-compReg LR + 1 baseline + 8 alternatives), write the comparison table to auc_results.csv, and verify expected reference values such as sc-compReg AUCs of 0.9802, 0.9972, 0.8124, 0.9157 across the four scenarios and the baseline AUC of 0.7561. The rationale is that only simulations with known differential-regulation truth allow objective, quantitative ranking of methods that otherwise lack labeled outcomes.

## Related tools

- **scCompReg** (Provides the likelihood-ratio differential-regulation statistic (with p-value computation and FDR control) whose AUC is being evaluated on the simulated datasets) — https://github.com/SUwonglab/sc-compReg
- **GENIE3** (GRN inference method used to build the alternative differential-network pipelines (GENIE3 + diffK/diffRank) benchmarked against sc-compReg)
- **PIDC** (GRN inference method used to build the alternative differential-network pipelines (PIDC + diffK/diffRank) benchmarked against sc-compReg)
- **sc-compReg repository data/prior_data** (Source of prior_data and example data obtained via git lfs clone, used in constructing the simulation and evaluation runs) — https://github.com/SUwonglab/sc-compReg

## Examples

```
require(devtools); devtools::install_github("SUwonglab/sc-compReg", ref="master", subdir="R_package")
```

## Evaluation signals

- sc-compReg likelihood-ratio AUCs match the reported values 0.9802, 0.9972, 0.8124, and 0.9157 across the four simulation scenarios
- scRNA-only baseline AUC matches the reported 0.7561
- auc_results.csv contains AUCs for all 10 methods (1 sc-compReg LR + 1 baseline + 8 GENIE3/PIDC × diffK/diffRank pipelines) across all 4 scenarios
- In the combined differential-regulation scenario, sc-compReg AUC (0.9157) substantially exceeds the scRNA-only baseline (0.7561)
- All methods are evaluated on identical simulated datasets with consistent cluster assignments to ensure a fair comparison

## Limitations

- AUC evaluation depends entirely on the fidelity of the simulated ground truth; performance on simulations may not transfer to real two-condition single-cell data
- Differential regulations discovered on real data require replication in additional patients and controls, which AUC-based simulation validation does not provide
- The sc-compReg method does not yet accommodate multiple donors per condition, constraining simulation designs to single-sample-per-condition comparisons

## Evidence

- [results] In the simulated scenario containing all three types of differential regulation, sc-compReg substantially outperforms the scRNA-only baseline method, achieving an AUC of 0.9157 versus 0.7561 for the baseline.: [evidence span withheld — anchor status: not_found]
- [methods] Simulation scenarios and reference AUCs: [evidence span withheld — anchor status: not_found]
- [intro] Simulation as validation rationale: "We will introduce methods for p-value computation and FDR control, and present simulation results to validate the methodology."
- [readme] Data acquisition for evaluation: "To download the data, make sure you have `git lfs` installed. Installation instructions can be found here: https://github.com/git-lfs/git-lfs/wiki/Installation"
