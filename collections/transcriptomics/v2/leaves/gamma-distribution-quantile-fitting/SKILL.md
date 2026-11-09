---
name: gamma-distribution-quantile-fitting
description: Use when computing p-values and FDR for likelihood ratio statistics testing
  differential regulation across two linked subpopulations and the standard Chi-square
  null approximation fails.
when_to_use_negative:
- When the Chi-square approximation to the LR null is empirically adequate for the
  test statistic at hand
- When only a handful of empirical LR scores are available, so that 10 stable lower
  quantiles cannot be extracted
- For p-value calibration of statistics other than the sc-compReg differential regulation
  likelihood ratio
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: sc-compReg
  role: Produces the empirical LR statistic scores for differential TF-TG regulatory
    relations that serve as the calibration target
  repo: https://github.com/SUwonglab/sc-compReg
- name: R (>= 3.6.0)
  role: Runtime environment in which the Gamma quantile function and the least-squares
    optimization (optim or nls) are executed
- name: scCompReg
  role: R package installed from source that implements the sc_compreg differential
    regulation test generating the LR scores
  repo: https://github.com/SUwonglab/sc-compReg
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-25089-2
    title: Sc-compReg enables the comparison of gene regulatory networks between conditions
      using single-cell data
  source_content_hash: sha256:6e8f7446359c114c950031f36b840f60c928443f8acfe4e4ad3649b053a5b8f1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SUwonglab/sc-compReg
  tools:
  - R (>= 3.6.0)
  - R (>=)
  - sc-compReg
  - scCompReg
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# gamma-distribution-quantile-fitting

## Summary

Calibrates the null distribution of the likelihood ratio (LR) statistic used by sc-compReg to test differential TF-TG regulatory relations by fitting a two-parameter Gamma distribution to the lower empirical quantiles of the LR scores, replacing the inadequate Chi-square approximation.

## When to use

Use when computing p-values and FDR for likelihood ratio statistics testing differential regulation across two linked subpopulations and the standard Chi-square null approximation fails.

## When NOT to use

- When the Chi-square approximation to the LR null is empirically adequate for the test statistic at hand
- When only a handful of empirical LR scores are available, so that 10 stable lower quantiles cannot be extracted
- For p-value calibration of statistics other than the sc-compReg differential regulation likelihood ratio

## Inputs

- Empirical LR statistic scores from the sc_compreg differential regulation test (numeric vector)
- R session (>= 3.6.0) with a numerical optimizer (optim or nls)

## Outputs

- Fitted Gamma shape (alpha) and scale (beta) parameters for the LR null distribution
- Residual sum of squares of the Gamma fit to the 10 target quantiles
- gamma_null_parameters.rds file storing the fitted parameters for downstream p-value/FDR computation

## How to apply

Load the empirical LR statistic scores produced by the sc_compreg differential regulation test into R (>= 3.6.0). Sort the LR scores and extract the lower 10 empirical quantiles t(i) as the calibration target. Fit a two-parameter Gamma distribution by minimizing the least-squares objective G(alpha,beta) = sum_i (q(i;alpha,beta) - t(i))^2, where q(i;alpha,beta) is the Gamma quantile function, using an R numerical optimizer such as optim or nls over alpha > 0 and beta > 0. The rationale is that the bulk of true null LRs populate the lower quantiles, so fitting there yields a valid null for assigning p-values and FDR to differential TF-TG relations where the Chi-square approximation is inadequate.

## Related tools

- **sc-compReg** (Produces the empirical LR statistic scores for differential TF-TG regulatory relations that serve as the calibration target) — https://github.com/SUwonglab/sc-compReg
- **R (>= 3.6.0)** (Runtime environment in which the Gamma quantile function and the least-squares optimization (optim or nls) are executed)
- **scCompReg** (R package installed from source that implements the sc_compreg differential regulation test generating the LR scores) — https://github.com/SUwonglab/sc-compReg

## Examples

```
fit <- optim(par=c(alpha=1,beta=1), fn=function(p) sum((qgamma(seq(0.05,0.5,0.05),shape=p[1],scale=p[2]) - t))^2, method="L-BFGS-B", lower=c(1e-8,1e-8)); saveRDS(fit$par,"gamma_null_parameters.rds")
```

## Evaluation signals

- Fitted Gamma quantiles q(i;alpha_hat,beta_hat) closely reproduce the 10 target empirical quantiles t(i)
- Residual sum of squares of the least-squares Gamma fit is small
- Estimated alpha and beta are both strictly positive
- Resulting p-values for known non-differential TF-TG pairs are approximately uniform under the fitted null
- Downstream FDR-controlled calls of differential regulatory relations are consistent with the fitted-null p-values

## Limitations

- Calibration depends on the lower 10 empirical quantiles; insufficient or noisy LR scores destabilize the fit
- The fitted null is specific to the sc_compReg LR statistic and data conditions; refitting is required for new datasets
- Fitting relies on a numerical optimizer and may converge to local minima over (alpha, beta)

## Evidence

- [intro] Null distribution via Gamma quantile fitting: "We will introduce methods for p-value computation and FDR control, and present simulation results to validate the methodology."
- [readme] R runtime requirement: "* Operating System: Linux or MacOS
* R (>= 3.6.0)"
- [abstract] sc-compReg role: "Here, we propose sc-compReg as a method for the comparative analysis of gene expression regulatory networks between two conditions"
- [intro] LR p-value and FDR computation target: "we present a new statistic for testing differential regulatory relations across two linked subpopulations"
