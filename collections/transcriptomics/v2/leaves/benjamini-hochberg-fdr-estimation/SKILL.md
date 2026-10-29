---
name: benjamini-hochberg-fdr-estimation
description: Use when testing differential regulatory relations between two conditions
  from paired single-cell data and needing calibrated p-values/FDR for the likelihood
  ratio statistic, especially when the standard Chi-square null appears anti-conservative.
when_to_use_negative:
- When testing differential regulation across multiple donors per condition — sc-compReg
  does not yet accommodate multiple donors per condition
- When the likelihood ratio statistics are already confirmed to be well calibrated
  under the asymptotic Chi-square null and no null simulation is available
- When the analysis is not comparative (single-condition regulatory inference, e.g.
  the PECA-style context-specific network setting) and no differential-regulation
  LR statistic exists
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: sc-compReg
  role: R package used to compute the likelihood ratio statistic for differential
    regulatory relations whose null distribution is simulated and used for p-value/FDR
    calibration
  repo: https://github.com/SUwonglab/sc-compReg
- name: R
  role: Runtime environment (>= 3.6.0) for the scCompReg package, Benjamini-Hochberg
    adjustment, Gamma fitting, and calibration plots
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-25089-2
    title: Sc-compReg enables the comparison of gene regulatory networks between conditions
      using single-cell data
  source_content_hash: sha256:24f48a4c80cac6aab6915b8594b773747d794906dd09f6b1aa2da45b684239dd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SUwonglab/sc-compReg
  tools:
  - R
  - sc-compReg
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# benjamini-hochberg-fdr-estimation

## Summary

Computes p-values and controls false discovery rate for the sc-compReg likelihood ratio statistic that tests differential regulatory relations between two linked subpopulations, comparing a Chi-square (df=3) asymptotic null against an empirically fitted Gamma null with Benjamini-Hochberg adjustment.

## When to use

Use when testing differential regulatory relations between two conditions from paired single-cell data and needing calibrated p-values/FDR for the likelihood ratio statistic, especially when the standard Chi-square null appears anti-conservative.

## When NOT to use

- When testing differential regulation across multiple donors per condition — sc-compReg does not yet accommodate multiple donors per condition
- When the likelihood ratio statistics are already confirmed to be well calibrated under the asymptotic Chi-square null and no null simulation is available
- When the analysis is not comparative (single-condition regulatory inference, e.g. the PECA-style context-specific network setting) and no differential-regulation LR statistic exists

## Inputs

- Likelihood ratio statistic distribution simulated under the sc-compReg null model of no differential regulation (from sc_compreg output or simulation)
- Nominal FDR threshold (0.1) for the Benjamini-Hochberg procedure
- Chi-square null parameters (df=3)
- Gamma distribution fitted to the simulated null LR statistics

## Outputs

- fdr_comparison.csv with per-null empirical FDR values (0.33 Chi-square vs 0.14 Gamma at nominal 0.1)
- fig2F_G.png calibration curves of nominal vs. empirical FDR for both nulls
- Benjamini-Hochberg adjusted p-values / significant differential regulatory relations

## How to apply

Generate or load simulated likelihood ratio statistics under the sc-compReg null model of no differential regulation between two linked subpopulations. First compute p-values under the standard asymptotic Chi-square null with df=3 and apply the Benjamini-Hochberg procedure at nominal FDR=0.1 (expected empirical FDR ~0.33, showing miscalibration). Then fit a Gamma distribution to the simulated LR statistic null, recompute p-values under this fitted null, and reapply Benjamini-Hochberg at nominal FDR=0.1 (expected empirical FDR ~0.14). Plot calibration curves of nominal vs. empirical FDR for both nulls (Fig. 2F-G), write the per-null empirical FDR values to fdr_comparison.csv, and save figures as fig2F_G.png. The rationale is that the Gamma-fitted null tracks the nominal FDR line more closely than the asymptotic Chi-square null.

## Related tools

- **sc-compReg** (R package used to compute the likelihood ratio statistic for differential regulatory relations whose null distribution is simulated and used for p-value/FDR calibration) — https://github.com/SUwonglab/sc-compReg
- **R** (Runtime environment (>= 3.6.0) for the scCompReg package, Benjamini-Hochberg adjustment, Gamma fitting, and calibration plots)

## Examples

```
pvals_bh <- p.adjust(pvals_chisq_df3, method = "BH"); sum(pvals_bh <= 0.1)  # empirical FDR at nominal 0.1
```

## Evaluation signals

- Empirical FDR at nominal FDR=0.1 under the Chi-square null should approximate the reported 0.33
- Empirical FDR at nominal FDR=0.1 under the Gamma-fitted null should approximate the reported 0.14
- The Gamma-fitted null calibration curve should track the nominal FDR line more closely than the Chi-square null curve (Fig. 2F-G)
- fdr_comparison.csv must contain per-null empirical FDR values matching the reported 0.14 and 0.33
- fig2F_G.png must be produced showing both null calibration curves

## Limitations

- The asymptotic Chi-square null (df=3) is miscalibrated for the LR statistic — empirical FDR (0.33) substantially exceeds the nominal 0.1 with Benjamini-Hochberg adjustment
- FDR calibration relies on simulation under the sc-compReg null model; the Gamma-fitted empirical FDR (0.14) still slightly exceeds nominal 0.1
- Differential regulations identified need to be replicated by additional patients and controls, and the method does not yet accommodate multiple donors per condition

## Evidence

- [intro] We will introduce methods for p-value computation and FDR control, and present simulation results to validate the methodology.: "We will introduce methods for p-value computation and FDR control, and present simulation results to validate the methodology."
- [readme] Install scCompReg from source: "Use the following command to install scCompReg R package from source code"
