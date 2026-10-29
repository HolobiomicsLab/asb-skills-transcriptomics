---
name: fdr-calibration-plotting
description: Use when testing differential regulatory relations across two linked
  single-cell subpopulations and you need to check whether the Chi-square null for
  the LR statistic is well calibrated or a Gamma-fitted null gives better FDR control.
when_to_use_negative:
- When the goal is p-value or FDR control for something other than the sc-compReg
  differential-regulation likelihood-ratio statistic (e.g., differential expression
  tests), since the Chi-square df=3 and Gamma-fitting calibration are specific to
  this statistic's null distribution
- When real, not null, differential regulations are the target — this skill validates
  null calibration, not discovery power
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: sc-compReg
  role: R package used to generate the LR statistic distribution under the sc-compReg
    null model and to compute p-values for differential regulatory relations
  repo: https://github.com/SUwonglab/sc-compReg
- name: R (>= 3.6.0)
  role: Runtime environment for running scCompReg, Benjamini-Hochberg adjustment,
    Gamma fitting, and calibration plotting
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-25089-2
    title: Sc-compReg enables the comparison of gene regulatory networks between conditions
      using single-cell data
  source_content_hash: sha256:1ca016a367197adb26357eef7feb0d1cbc97aa31167f7ebbfe687e1a53c9b048
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SUwonglab/sc-compReg
  tools:
  - R (>= 3.6.0)
  - sc-compReg
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# fdr-calibration-plotting

## Summary

Empirically calibrates the false discovery rate of the sc-compReg likelihood-ratio statistic for differential regulation by comparing p-values computed under the standard asymptotic Chi-square (df=3) null against a Gamma-fitted empirical null, then plotting nominal vs. empirical FDR calibration curves (Fig. 2F-G).

## When to use

Use when testing differential regulatory relations across two linked single-cell subpopulations and you need to check whether the Chi-square null for the LR statistic is well calibrated or a Gamma-fitted null gives better FDR control.

## When NOT to use

- When the goal is p-value or FDR control for something other than the sc-compReg differential-regulation likelihood-ratio statistic (e.g., differential expression tests), since the Chi-square df=3 and Gamma-fitting calibration are specific to this statistic's null distribution
- When real, not null, differential regulations are the target — this skill validates null calibration, not discovery power

## Inputs

- Simulated or observed likelihood-ratio statistics for differential regulatory relations from the sc-compReg null model
- Null-model simulation output from the sc-compReg R package
- Nominal FDR level (e.g., 0.1) for the Benjamini-Hochberg procedure

## Outputs

- fdr_comparison.csv containing per-null empirical FDR values (Chi-square vs. Gamma)
- fig2F_G.png with nominal vs. empirical FDR calibration curves for both nulls
- Empirical FDR estimates at nominal FDR=0.1 (~0.33 Chi-square, ~0.14 Gamma)

## How to apply

Simulate (or load) the likelihood-ratio statistic distribution under the sc-compReg null model of no differential regulation between the two linked subpopulations using the sc-compReg R package. Compute p-values under the standard asymptotic Chi-square null with df=3, apply Benjamini-Hochberg at nominal FDR=0.1, and record the empirical FDR (expected ~0.33, showing mis-calibration). Then fit a Gamma distribution to the simulated null LR statistics, recompute p-values and BH-adjusted FDR at 0.1 (expected ~0.14, showing better calibration). Plot nominal vs. empirical FDR calibration curves for both nulls, write per-null empirical FDR values to fdr_comparison.csv, and save the curves as fig2F_G.png. The Gamma-fitted null is preferred because its calibration curve tracks the nominal diagonal more closely than the Chi-square null.

## Related tools

- **sc-compReg** (R package used to generate the LR statistic distribution under the sc-compReg null model and to compute p-values for differential regulatory relations) — https://github.com/SUwonglab/sc-compReg
- **R (>= 3.6.0)** (Runtime environment for running scCompReg, Benjamini-Hochberg adjustment, Gamma fitting, and calibration plotting)

## Examples

```
compreg.output = sc_compreg(sample1$O1, sample1$E1, sample1$O1.idx, sample1$E1.idx, sample1$symbol1, sample1$peak.name1, sample2$O2, sample2$E2, ...)
```

## Evaluation signals

- Empirical FDR at nominal FDR=0.1 under the Chi-square null matches the reported value of 0.33
- Empirical FDR at nominal FDR=0.1 under the Gamma-fitted null matches the reported value of 0.14
- Gamma-fitted null calibration curve tracks the nominal (diagonal) line more closely than the Chi-square null curve, as in Fig. 2F-G
- fdr_comparison.csv is written with both per-null empirical FDR values and fig2F_G.png is saved

## Limitations

- The standard asymptotic Chi-square (df=3) null is mis-calibrated for this LR statistic, inflating empirical FDR well above nominal; using it without Gamma fitting can overstate significance
- The empirical FDR values (0.33, 0.14) are specific to the simulated null configuration and may vary with simulation settings

## Evidence

- [intro] We will introduce methods for p-value computation and FDR control, and present simulation results to validate the methodology.: "We will introduce methods for p-value computation and FDR control, and present simulation results to validate the methodology."
- [results] the empirical false discovery rate was 0.33 for the Chi-square method versus 0.14 for the Gamma method, showing the Gamma null is better calibrated.: [evidence span withheld — anchor status: not_found]
- [readme] devtools::install_github("SUwonglab/sc-compReg", ref="master", subdir="R_package"): "Use the following command to install scCompReg R package from source code:"
