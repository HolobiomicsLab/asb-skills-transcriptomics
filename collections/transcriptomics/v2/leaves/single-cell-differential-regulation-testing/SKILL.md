---
name: single-cell-differential-regulation-testing
description: Use when sc_compreg has produced empirical likelihood ratio scores for
  TF-TG differential regulation and you need calibrated p-values and FDR control for
  them.
when_to_use_negative:
- You only need coupled clustering/embedding or prior peak-gene network construction
  — this skill covers only the p-value/FDR calibration of the LR statistic
- The analysis targets a single condition's regulatory network (e.g., PECA-style context-specific
  inference) rather than differential relations across two conditions
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: sc-compReg
  role: Produces the likelihood ratio statistics for differential TF-TG regulation
    whose null distribution is calibrated here; provides the sc_compreg function and
    scCompReg R package
  repo: https://github.com/SUwonglab/sc-compReg
- name: scCompReg
  role: R package installed from source that must be loaded to run sc_compreg and
    access the LR scores
  repo: https://github.com/SUwonglab/sc-compReg
- name: R (>= 3.6.0)
  role: Runtime in which the LR scores are sorted, the lower 10 empirical quantiles
    are extracted, and the Gamma fit is optimized (optim or nls)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-25089-2
    title: Sc-compReg enables the comparison of gene regulatory networks between conditions
      using single-cell data
  source_content_hash: sha256:8db549324d82d3ad8aeca72eccc58d78685030e87a041d2ffa12a083e0961a18
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
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# single-cell-differential-regulation-testing

## Summary

Computes p-values and FDR for the likelihood ratio statistic testing differential TF-TG regulatory relations between two conditions in sc-compReg, by calibrating the null with a fitted Gamma distribution instead of the failing Chi-square approximation.

## When to use

Use when sc_compreg has produced empirical likelihood ratio scores for TF-TG differential regulation and you need calibrated p-values and FDR control for them.

## When NOT to use

- You only need coupled clustering/embedding or prior peak-gene network construction — this skill covers only the p-value/FDR calibration of the LR statistic
- The analysis targets a single condition's regulatory network (e.g., PECA-style context-specific inference) rather than differential relations across two conditions

## Inputs

- Empirical LR statistic scores from the sc_compreg differential regulation test (e.g., compreg.output LR scores)
- R (>= 3.6.0) environment with the scCompReg package installed

## Outputs

- Fitted Gamma null parameters alpha and beta (gamma_null_parameters.rds)
- Residual sum of squares of the Gamma fit
- Calibrated p-values and FDR-controlled results for differential TF-TG regulatory relations

## How to apply

Load the empirical LR statistic scores from the sc_compreg differential regulation test into R (>= 3.6.0). Sort the LR scores and extract the lower 10 empirical quantiles t(i) as the calibration target, since the standard Chi-square null approximation fails. Fit a two-parameter Gamma distribution by minimizing the least-squares objective G(alpha,beta) = sum_i (q(i;alpha,beta) - t(i))^2 over alpha > 0 and beta > 0, where q(i;alpha,beta) is the Gamma quantile function, using an R numerical optimizer such as optim or nls. Record the fitted alpha and beta estimates and the residual sum of squares, then save the parameters (e.g., as gamma_null_parameters.rds) for downstream p-value computation and FDR control.

## Related tools

- **sc-compReg** (Produces the likelihood ratio statistics for differential TF-TG regulation whose null distribution is calibrated here; provides the sc_compreg function and scCompReg R package) — https://github.com/SUwonglab/sc-compReg
- **scCompReg** (R package installed from source that must be loaded to run sc_compreg and access the LR scores) — https://github.com/SUwonglab/sc-compReg
- **R (>= 3.6.0)** (Runtime in which the LR scores are sorted, the lower 10 empirical quantiles are extracted, and the Gamma fit is optimized (optim or nls))

## Examples

```
library(scCompReg); compreg.output = sc_compreg(sample1$O1, sample1$E1, sample1$O1.idx, sample1$E1.idx, sample1$symbol1, sample1$peak.name1, sample2$O2, sample2$E2, ...)
```

## Evaluation signals

- The fitted Gamma quantiles q(i;alpha_hat,beta_hat) closely reproduce the 10 target empirical quantiles t(i) (small residual sum of squares)
- alpha_hat > 0 and beta_hat > 0 after optimization convergence
- Fitted parameters are saved to gamma_null_parameters.rds and can be reloaded via readRDS
- Downstream p-values computed from the fitted Gamma null are well-calibrated (e.g., uniform under the null in simulations, as validated in the paper)

## Limitations

- The Chi-square null approximation fails for the LR statistic, so any shortcut using it instead of the Gamma calibration is invalid
- Differential regulations discovered by this procedure need replication in additional patients and controls
- The method does not yet accommodate multiple donors per condition; comparison is currently limited to one donor per condition

## Evidence

- [full_text] The null distribution of the likelihood ratio is calibrated by fitting a Gamma distribution to the lower quantiles of the empirical likelihood ratios, rather than using the Chi-square approximation.: [evidence span withheld — anchor status: not_found]
- [intro] We will introduce methods for p-value computation and FDR control, and present simulation results to validate the methodology.: "We will introduce methods for p-value computation and FDR control, and present simulation results to validate the methodology."
- [readme] Use the following command to install scCompReg R package from source code:: "Use the following command to install scCompReg R package from source code:"
- [methods] compreg.output = sc_compreg(sample1$O1,: "compreg.output = sc_compreg(sample1$O1,"
- [readme] Operating System: Linux or MacOS * R (>= 3.6.0): "Operating System: Linux or MacOS
* R (>= 3.6.0)"
