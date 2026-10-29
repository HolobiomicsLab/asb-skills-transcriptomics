---
name: gamma-distribution-fitting-to-null-statistics
description: Use when testing differential regulatory relations between two linked
  single-cell subpopulations and the asymptotic Chi-square null for the likelihood-ratio
  statistic is suspected to be miscalibrated, i.e. empirical FDR exceeds nominal FDR.
when_to_use_negative:
- When the likelihood-ratio statistic's null distribution is already empirically well
  calibrated (Chi-square empirical FDR tracks nominal FDR), so a Gamma fit adds no
  benefit
- When the analysis is not a two-condition differential regulatory test with the sc-compReg
  LR statistic (e.g. bulk-only differential expression), where this null calibration
  was not established
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: sc-compReg
  role: R package used to simulate LR statistics under the sc-compReg null model and
    compute p-values/FDR for differential regulatory relations
  repo: https://github.com/SUwonglab/sc-compReg
- name: R (>= 3.6.0)
  role: Runtime environment for fitting the Gamma null, running Benjamini-Hochberg,
    and generating calibration plots
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-25089-2
    title: Sc-compReg enables the comparison of gene regulatory networks between conditions
      using single-cell data
  source_content_hash: sha256:d389c8df51c9aa6ccb22ab15e5acd28e36d507546587780f64a43508afbc2190
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SUwonglab/sc-compReg
  tools:
  - R (>= 3.6.0)
  - sc-compReg
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# gamma-distribution-fitting-to-null-statistics

## Summary

Empirically calibrates the null distribution of the sc-compReg likelihood-ratio statistic for testing differential regulatory relations by fitting a Gamma distribution to simulated null statistics, in place of the asymptotic Chi-square (df=3) null, and validates calibration via Benjamini-Hochberg FDR.

## When to use

Use when testing differential regulatory relations between two linked single-cell subpopulations and the asymptotic Chi-square null for the likelihood-ratio statistic is suspected to be miscalibrated, i.e. empirical FDR exceeds nominal FDR.

## When NOT to use

- When the likelihood-ratio statistic's null distribution is already empirically well calibrated (Chi-square empirical FDR tracks nominal FDR), so a Gamma fit adds no benefit
- When the analysis is not a two-condition differential regulatory test with the sc-compReg LR statistic (e.g. bulk-only differential expression), where this null calibration was not established

## Inputs

- Simulated or observed likelihood-ratio statistics from the sc-compReg null model (no differential regulation between two linked subpopulations)
- Nominal FDR threshold (Benjamini-Hochberg procedure at 0.1)
- Chi-square null with df=3 for baseline comparison

## Outputs

- fdr_comparison.csv with per-null empirical FDR values (Chi-square ~0.33 vs Gamma ~0.14 at nominal FDR=0.1)
- Calibration curves of nominal vs. empirical FDR for both nulls (fig2F_G.png)
- Gamma-fitted null p-values for the LR statistics

## How to apply

Generate or load the likelihood-ratio statistic distribution under the sc-compReg null model of no differential regulation between the two linked subpopulations (simulated via the sc-compReg R package). Compute p-values under the standard asymptotic Chi-square null with df=3 and apply Benjamini-Hochberg at nominal FDR=0.1 to obtain the empirical FDR (expected ~0.33, showing poor calibration). Then fit a Gamma distribution to the simulated null LR statistics, recompute p-values under this fitted null, and re-apply Benjamini-Hochberg at nominal FDR=0.1 (expected empirical FDR ~0.14). Plot calibration curves of nominal vs. empirical FDR for both nulls, as in Fig. 2F-G, and write per-null empirical FDR values to fdr_comparison.csv with figures saved as fig2F_G.png. The rationale is that the Gamma-fitted null better matches the empirical distribution of the LR statistic, keeping the BH-adjusted false discovery rate closer to the nominal level.

## Related tools

- **sc-compReg** (R package used to simulate LR statistics under the sc-compReg null model and compute p-values/FDR for differential regulatory relations) — https://github.com/SUwonglab/sc-compReg
- **R (>= 3.6.0)** (Runtime environment for fitting the Gamma null, running Benjamini-Hochberg, and generating calibration plots)

## Examples

```
devtools::install_github("SUwonglab/sc-compReg", ref="master", subdir="R_package"); compreg.output = sc_compreg(sample1$O1, sample1$E1, sample1$O1.idx, sample1$E1.idx, sample1$symbol1, sample1$peak.name1, sample2$O2, sample2$E2, ...)
```

## Evaluation signals

- At nominal FDR=0.1 (Benjamini-Hochberg adjusted), the empirical FDR should be ~0.33 under the Chi-square df=3 null and ~0.14 under the Gamma-fitted null
- The Gamma-null calibration curve should track the nominal line (y=x) more closely than the Chi-square curve, as in Fig. 2F-G
- fdr_comparison.csv must contain both empirical FDR values matching the reported numbers
- fig2F_G.png must render both calibration curves for visual comparison

## Limitations

- The Chi-square (df=3) asymptotic null is anti-conservative for this LR statistic: nominal FDR=0.1 yields empirical FDR ~0.33, so p-values based on it overstate significance
- Calibration was demonstrated via simulation under the sc-compReg null model; the fitted Gamma null is specific to this statistic and simulation setting and may not transfer to other test statistics
- In the CLL application, the differential regulations discovered would need to be replicated by additional patients and controls

## Evidence

- [results] At an estimated FDR of 0.1, the empirical false discovery rate was 0.33 for the Chi-square method versus 0.14 for the Gamma method, showing the Gamma null is better calibrated.: [evidence span withheld — anchor status: not_found]
- [intro] We will introduce methods for p-value computation and FDR control, and present simulation: [evidence span withheld — anchor status: not_found]
- [readme] Use the following command to install scCompReg R package from source code:: "Use the following command to install scCompReg R package from source code:"
