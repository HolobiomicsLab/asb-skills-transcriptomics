---
name: pvalue-computation-and-fdr-control
description: Use when p-values/FDR are needed for sc_compreg likelihood ratio outputs
  from paired scRNA-seq + scATAC-seq data and the Chi-square null approximation to
  the LR statistic is inadequate.
when_to_use_negative:
- Analysis is a single-condition regulatory network inference (e.g., PECA tissue-specific
  network) rather than a two-condition differential test producing LR scores
- The likelihood ratio null can already be adequately approximated by a Chi-square
  distribution for the statistic at hand — the Gamma quantile calibration is needed
  precisely because the Chi-square approximation fails
- Differential relations are being tested across multiple donors per condition — the
  article notes sc-compReg does not yet accommodate multiple donors per condition
tools:
- name: sc-compReg
  role: Produces the LR statistic scores whose null distribution is calibrated by
    the Gamma quantile fit for p-value/FDR computation
  repo: https://github.com/SUwonglab/sc-compReg
- name: scCompReg
  role: R package (installed from source, R >= 3.6.0) providing the sc_compreg function
    that generates the empirical LR scores and in which the Gamma fitting is performed
  repo: https://github.com/SUwonglab/sc-compReg
- name: R (>= 3.6.0)
  role: Runtime environment with numerical optimizers (optim/nls) used to minimize
    the Gamma quantile least-squares objective
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-25089-2
    title: Sc-compReg enables the comparison of gene regulatory networks between conditions
      using single-cell data
  source_content_hash: sha256:63c5a4f98994c411ab543eeb35815f925da54d2e384268419a1185876c99dbef
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# pvalue-computation-and-fdr-control

## Summary

Computes p-values and FDR for the sc-compReg likelihood ratio statistic testing differential TF-TG regulatory relations by fitting a two-parameter Gamma distribution to the lower 10 empirical quantiles of the LR scores instead of using the failing Chi-square null approximation.

## When to use

Use when p-values/FDR are needed for sc_compreg likelihood ratio outputs from paired scRNA-seq + scATAC-seq data and the Chi-square null approximation to the LR statistic is inadequate.

## When NOT to use

- Analysis is a single-condition regulatory network inference (e.g., PECA tissue-specific network) rather than a two-condition differential test producing LR scores
- The likelihood ratio null can already be adequately approximated by a Chi-square distribution for the statistic at hand — the Gamma quantile calibration is needed precisely because the Chi-square approximation fails
- Differential relations are being tested across multiple donors per condition — the article notes sc-compReg does not yet accommodate multiple donors per condition

## Inputs

- Empirical LR statistic scores from sc_compreg differential regulation testing (compreg.output)
- scRNA-seq and scATAC-seq sample matrices (sample1.rds, sample2.rds) used to run sc_compreg
- motif file and peak-gene prior data from prior_data directory (e.g., motif_human.rds)

## Outputs

- Fitted Gamma null parameters (alpha_hat, beta_hat) saved as gamma_null_parameters.rds
- Residual sum of squares of the Gamma quantile fit
- p-values and FDR-controlled calls of differential TF-TG regulatory relations

## How to apply

Load the empirical LR statistic scores produced by the sc_compreg differential regulation test into R (>= 3.6.0), then sort them and extract the lower 10 empirical quantiles t(i) as the calibration target. Fit a two-parameter Gamma distribution by minimizing the least-squares objective G(alpha,beta) = sum_i (q(i;alpha,beta) - t(i))^2, where q(i;alpha,beta) is the Gamma quantile function, using an R optimizer such as optim or nls constrained to alpha > 0 and beta > 0. Record the fitted alpha and beta estimates and the residual sum of squares, then save the fitted parameters (e.g., gamma_null_parameters.rds) for downstream p-value computation and FDR control. The rationale is that the Gamma fit calibrated to the empirical LR lower tail reproduces the null distribution better than the Chi-square approximation.

## Related tools

- **sc-compReg** (Produces the LR statistic scores whose null distribution is calibrated by the Gamma quantile fit for p-value/FDR computation) — https://github.com/SUwonglab/sc-compReg
- **scCompReg** (R package (installed from source, R >= 3.6.0) providing the sc_compreg function that generates the empirical LR scores and in which the Gamma fitting is performed) — https://github.com/SUwonglab/sc-compReg
- **R (>= 3.6.0)** (Runtime environment with numerical optimizers (optim/nls) used to minimize the Gamma quantile least-squares objective)

## Examples

```
fit <- optim(c(alpha=1,beta=1), function(p) sum((qgamma(tq, shape=p[1], scale=p[2]) - t)^2), control=list(parscale=c(1,1))); saveRDS(fit, 'gamma_null_parameters.rds')
```

## Evaluation signals

- Fitted Gamma quantiles q(i;alpha_hat,beta_hat) closely reproduce the 10 target empirical quantiles t(i), i.e., small residual sum of squares
- alpha_hat > 0 and beta_hat > 0 after optimizer convergence over the constrained parameter space
- Fitted parameters are successfully serialized to gamma_null_parameters.rds and reloadable with readRDS
- Downstream p-values computed from the Gamma null are calibrated such that FDR control is valid on the differential TF-TG calls

## Limitations

- The calibration uses only the lower 10 empirical quantiles, so the Gamma fit is anchored to the null lower tail and depends on sufficient empirical LR scores being available
- Standard Chi-square null approximation fails for this LR statistic and must not be substituted for the Gamma fit
- Differential regulations discovered in the CLL analysis were not yet replicated in additional patients/controls
- sc-compReg does not yet support multiple donors per condition or bulk HiChIP data in the comparison

## Evidence

- [intro] LR p-value null calibration: "We will introduce methods for p-value computation and FDR control, and present simulation results to validate the methodology."
- [other] Gamma quantile fit procedure: [evidence span withheld — anchor status: not_found]
- [other] lower 10 quantiles calibration: [evidence span withheld — anchor status: not_found]
- [readme] environment requirement: "* Operating System: Linux or MacOS
* R (>= 3.6.0)"
