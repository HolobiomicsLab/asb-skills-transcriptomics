---
name: chi-square-asymptotic-null-comparison
description: Use when computing p-values and FDR for sc-compReg differential-regulation
  LR statistics and you need to decide whether the asymptotic Chi-square null is miscalibrated
  relative to a fitted (Gamma) empirical null.
when_to_use_negative:
- The goal is differential regulatory analysis on real data (network construction),
  not validation of the p-value/FDR machinery for the LR statistic
- The LR statistic null is already known to be well calibrated in your setting — the
  comparison only matters when the asymptotic Chi-square approximation is suspect
- Input is a single-cell expression/accessibility dataset rather than a simulated
  or null-model distribution of LR statistics
edam_operation: http://edamontology.org/operation_2239
tools:
- name: sc-compReg
  role: R package used to generate the LR statistic null distribution for differential
    regulatory relations between two linked subpopulations
  repo: https://github.com/SUwonglab/sc-compReg
- name: R (>= 3.6.0)
  role: Runtime environment for pchisq/pgamma p-value computation, Gamma null fitting,
    and Benjamini-Hochberg FDR adjustment
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-25089-2
    title: Sc-compReg enables the comparison of gene regulatory networks between conditions
      using single-cell data
  source_content_hash: sha256:5a6d120da1001e92d59d361c4fb582037e307bd809bf88a98b1bba1f8d1de1bb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SUwonglab/sc-compReg
  tools:
  - R (>= 3.6.0)
  - sc-compReg
  edam_operation: http://edamontology.org/operation_2239
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# chi-square-asymptotic-null-comparison

## Summary

Empirically calibrates the null distribution of the sc-compReg likelihood ratio (LR) statistic for differential regulatory relations by comparing p-values from the standard asymptotic Chi-square null against p-values from an empirically fitted Gamma null, then assessing which null yields accurate Benjamini-Hochberg FDR control.

## When to use

Use when computing p-values and FDR for sc-compReg differential-regulation LR statistics and you need to decide whether the asymptotic Chi-square null is miscalibrated relative to a fitted (Gamma) empirical null.

## When NOT to use

- The goal is differential regulatory analysis on real data (network construction), not validation of the p-value/FDR machinery for the LR statistic
- The LR statistic null is already known to be well calibrated in your setting — the comparison only matters when the asymptotic Chi-square approximation is suspect
- Input is a single-cell expression/accessibility dataset rather than a simulated or null-model distribution of LR statistics

## Inputs

- Simulated likelihood-ratio statistic distribution under the sc-compReg null of no differential regulation between two linked subpopulations
- Nominal FDR level (0.1) and Chi-square degrees of freedom (df=3)
- Fitted Gamma distribution parameters for the null LR statistics

## Outputs

- fdr_comparison.csv with per-null empirical FDR values (0.33 Chi-square vs 0.14 Gamma at nominal FDR 0.1)
- fig2F_G.png calibration curves of nominal vs empirical FDR for both nulls
- BH-adjusted p-values for each null distribution

## How to apply

Simulate (or load) the LR statistic distribution under the sc-compReg null model of no differential regulation between two linked subpopulations, using the sc-compReg R package (R >= 3.6.0, installable via devtools::install_github('SUwonglab/sc-compReg', ref='master', subdir='R_package')). Compute p-values under the asymptotic Chi-square null with df=3, apply Benjamini-Hochberg at nominal FDR=0.1, and record the empirical false discovery rate (reported value: 0.33). Fit a Gamma distribution to the simulated null LR statistics, recompute p-values, and repeat the BH procedure at nominal FDR=0.1 (reported value: 0.14). Plot nominal-vs-empirical FDR calibration curves for both nulls (Fig. 2F-G panels); accept the run when the Gamma-fitted null curve tracks the nominal diagonal more closely than the Chi-square null.

## Related tools

- **sc-compReg** (R package used to generate the LR statistic null distribution for differential regulatory relations between two linked subpopulations) — https://github.com/SUwonglab/sc-compReg
- **R (>= 3.6.0)** (Runtime environment for pchisq/pgamma p-value computation, Gamma null fitting, and Benjamini-Hochberg FDR adjustment)

## Examples

```
pvals = pchisq(LR, df=3, lower.tail=FALSE); padj = p.adjust(pvals, method='BH'); empirical_fdr = sum(padj <= 0.1 & is_null) / sum(padj <= 0.1)
```

## Evaluation signals

- Empirical FDR at nominal FDR=0.1 equals ~0.33 for the Chi-square (df=3) null and ~0.14 for the Gamma-fitted null
- Gamma-fitted null calibration curve lies closer to the nominal diagonal than the Chi-square null curve (Fig. 2F-G criterion)
- BH-adjusted p-values are computable for all simulated null LR statistics under both nulls
- fdr_comparison.csv contains per-null empirical FDR values and fig2F_G.png renders both calibration curves

## Limitations

- The empirical FDR values (0.33, 0.14) are simulation-specific; they depend on the simulated null LR distribution and may differ with other settings
- The Chi-square miscalibration is characterized at the single nominal level (0.1) in the reproduced check; full calibration across all FDR levels requires the full Fig. 2F-G curve comparison
- sc-compReg requires R (>= 3.6.0) plus Bedtools and Homer on Linux, and git lfs to fetch the data directory

## Evidence

- [intro] p-value computation and FDR control objective: "We will introduce methods for p-value computation and FDR control, and present simulation results to validate the methodology."
- [other] task card workflow with df=3 and empirical FDR values: [evidence span withheld — anchor status: not_found]
- [other] reported empirical FDR contrast: [evidence span withheld — anchor status: not_found]
- [readme] package installation command: "devtools::install_github("SUwonglab/sc-compReg", ref="master", subdir="R_package")"
