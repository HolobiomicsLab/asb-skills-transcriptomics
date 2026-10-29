---
name: likelihood-ratio-statistic-null-calibration
description: Use when computing p-values/FDR for likelihood ratio statistics from
  a differential regulatory relation test between two linked subpopulations, and the
  standard Chi-square null is suspected to be anti-conservative or miscalibrated.
when_to_use_negative:
- Input is raw scRNA-seq/scATAC-seq count matrices rather than computed LR statistics
  — run sc_compreg first to obtain the differential regulation likelihood ratios
- The LR statistic already has a validated asymptotic null distribution for the statistic
  at hand and no calibration check is needed
- Analysis involves more than two conditions or multiple donors per condition — sc-compReg
  does not yet accommodate multiple donors under each of the two conditions
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: sc-compReg
  role: R package whose differential regulation test produces the likelihood ratio
    statistics whose null distribution is calibrated here
  repo: https://github.com/SUwonglab/sc-compReg
- name: R (>= 3.6.0)
  role: Runtime environment used for quantile extraction, Gamma quantile fitting via
    optim/nls, p-value computation, and Benjamini-Hochberg FDR adjustment
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-25089-2
    title: Sc-compReg enables the comparison of gene regulatory networks between conditions
      using single-cell data
  source_content_hash: sha256:944efea4a9b069bfc9dd01e07aae9d2cce0c506f0f23bc3caaf8befea41621f9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SUwonglab/sc-compReg
  tools:
  - R (>= 3.6.0)
  - R (>=)
  - sc-compReg
  - scCompReg
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# likelihood-ratio-statistic-null-calibration

## Summary

Calibrates the null distribution of the sc-compReg likelihood ratio (LR) statistic testing differential TF-TG regulatory relations by fitting a Gamma distribution to the lower 10 empirical quantiles of the null LR scores, replacing the miscalibrated asymptotic Chi-square (df=3) null for p-value and FDR computation.

## When to use

Use when computing p-values/FDR for likelihood ratio statistics from a differential regulatory relation test between two linked subpopulations, and the standard Chi-square null is suspected to be anti-conservative or miscalibrated.

## When NOT to use

- Input is raw scRNA-seq/scATAC-seq count matrices rather than computed LR statistics — run sc_compreg first to obtain the differential regulation likelihood ratios
- The LR statistic already has a validated asymptotic null distribution for the statistic at hand and no calibration check is needed
- Analysis involves more than two conditions or multiple donors per condition — sc-compReg does not yet accommodate multiple donors under each of the two conditions

## Inputs

- Empirical LR statistic scores from the sc_compreg differential regulation test (R numeric vector)
- Nominal FDR threshold (Benjamini-Hochberg, e.g., 0.1)
- Chi-square null degrees of freedom (df=3) for the comparison baseline

## Outputs

- Fitted Gamma null parameters (alpha, beta) and fit residual sum of squares, e.g., gamma_null_parameters.rds
- P-values for the LR statistics under the Gamma null (and optionally the Chi-square df=3 null)
- Benjamini-Hochberg-adjusted FDR calls at nominal FDR=0.1 and per-null empirical FDR values (e.g., fdr_comparison.csv)
- Calibration curves of nominal vs. empirical FDR for both nulls (e.g., fig2F_G.png, reproducing Fig. 2F-G)

## How to apply

Load the empirical LR statistic scores produced by the sc_compreg differential regulation test into R (>= 3.6.0), sort them, and extract the lower 10 empirical quantiles t(i) as the calibration target. Fit a two-parameter Gamma distribution by minimizing the least-squares objective G(alpha,beta) = sum_i (q(i;alpha,beta) - t(i))^2 over alpha > 0, beta > 0 using an R optimizer such as optim or nls, and record the fitted parameters and residual sum of squares. For comparison, compute p-values under the asymptotic Chi-square null with df=3 and apply Benjamini-Hochberg at nominal FDR=0.1, then repeat under the fitted Gamma null. The rationale is that the Chi-square approximation is poorly calibrated (empirical FDR 0.33 at nominal 0.1) while the Gamma-fitted null tracks the nominal line closely (empirical FDR 0.14), as shown in Fig. 2F-G of the paper. Save fitted parameters (e.g., gamma_null_parameters.rds) and, if validating, write per-null empirical FDR values to fdr_comparison.csv and plot calibration curves.

## Related tools

- **sc-compReg** (R package whose differential regulation test produces the likelihood ratio statistics whose null distribution is calibrated here) — https://github.com/SUwonglab/sc-compReg
- **R (>= 3.6.0)** (Runtime environment used for quantile extraction, Gamma quantile fitting via optim/nls, p-value computation, and Benjamini-Hochberg FDR adjustment)

## Examples

```
fit <- optim(c(1,1), function(p) sum((qgamma(1:10/11, p[1], scale=1/p[2]) - t)^2), method="L-BFGS-B"); saveRDS(fit$par, 'gamma_null_parameters.rds')
```

## Evaluation signals

- At nominal FDR=0.1 (Benjamini-Hochberg), the empirical FDR is approximately 0.33 for the Chi-square (df=3) null and approximately 0.14 for the Gamma-fitted null
- The fitted Gamma quantiles q(i;alpha_hat,beta_hat) closely reproduce the 10 target lower empirical quantiles t(i) (small residual sum of squares)
- The Gamma-fitted null calibration curve tracks the nominal FDR line more closely than the Chi-square null curve, as in Fig. 2F-G
- Fitted parameters satisfy alpha > 0 and beta > 0, and the fit residuals and parameters are persisted (e.g., gamma_null_parameters.rds)
- Empirical FDR values for both nulls are written to fdr_comparison.csv and calibration figures saved as fig2F_G.png

## Limitations

- The Gamma null is calibrated to the lower 10 empirical quantiles of the simulated LR null scores; calibration depends on having an adequate sample of null statistics
- The Chi-square (df=3) asymptotic null is anti-conservative for this statistic, yielding an empirical FDR of 0.33 at nominal 0.1, so it should not be trusted for this test
- sc-compReg does not yet accommodate multiple donors under each of the two conditions, limiting the scope of null calibration to single-sample-per-condition comparisons
- Differential regulations discovered this way will need to be replicated by additional patients and controls

## Evidence

- [intro] Intro motivation for p-value/FDR methods for the LR statistic: "We will introduce methods for p-value computation and FDR control, and present simulation results to validate the methodology."
- [intro] Statistic for testing differential regulatory relations across linked subpopulations: "we present a new statistic for testing differential regulatory relations across two linked subpopulations"
- [readme] scCompReg is the R package producing the LR statistics: "scCompReg (**S**ingle-**C**ell **C**omparative **R**egulatory analysis) is an R package that provides coupled clustering and joint embedding of scRNA-seq and scATAC-seq on one sample"
- [readme] Software installation context for the R environment used in calibration: "devtools::install_github("SUwonglab/sc-compReg", ref="master", subdir="R_package")"
