---
name: negative-binomial-gaussian-copula-count-simulation
description: Use when evaluating or benchmarking gene co-expression estimators (CS-CORE,
  rho-sctransform, locCSN, etc.) on synthetic UMI counts where a known true latent
  correlation must be held fixed across a grid of expression levels.
when_to_use_negative:
- When the goal is differential co-expression testing on real data rather than estimator
  benchmarking under known ground truth
- When the underlying generative model must differ from the Poisson measurement process,
  since the paper's measurement model uses only a Poisson distribution and defers
  negative-binomial count modeling to future work
edam_operation: http://edamontology.org/operation_0289
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: CS-CORE / CSCORE (R package)
  role: Estimator under evaluation; run as CSCORE/CSCORE_IRLS with IRLS estimation
    and post_process_est on each simulated dataset to estimate co-expressions and
    check mean-correlation bias
  repo: https://github.com/ChangSuBiostats/CS-CORE
- name: sctransform
  role: Competing co-expression estimator (rho-sctransform) evaluated on the same
    simulated UMI counts
- name: locCSN
  role: Competing co-expression estimator evaluated for bias and MSE on simulated
    gene-pair counts
  repo: https://github.com/xuranw/locCSN
- name: NoiseRegularization
  role: Competing co-expression estimator evaluated for bias and MSE on simulated
    gene-pair counts
  repo: https://github.com/RuoyuZhang/NoiseRegularization
- name: normalisr
  role: Competing co-expression estimator evaluated for bias and MSE on simulated
    gene-pair counts
  repo: https://github.com/lingfeiwang/normalisr
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-40503-7
    title: Cell-type-specific co-expression inference from single cell RNA-sequencing
      data
  source_content_hash: sha256:f84ca4f8fc2c06c30477deada1729e500b7375c18157c1f96196ae1af407d006
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ChangSuBiostats/CS-CORE
  tools:
  - CS-CORE
  - CS-CORE / CSCORE (R package)
  - CSCORE (R package)
  - NoiseRegularization
  - locCSN
  - normalisr
  - sctransform
  edam_operation: http://edamontology.org/operation_0289
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# negative-binomial-gaussian-copula-count-simulation

## Summary

A scRNA-seq simulation scheme that generates synthetic gene-pair UMI counts with negative-binomial marginals and a Gaussian-copula latent correlation (e.g., rho = 0.5), used to benchmark co-expression estimators for mean-correlation bias and MSE against CS-CORE-style ground truth.

## When to use

Use when evaluating or benchmarking gene co-expression estimators (CS-CORE, rho-sctransform, locCSN, etc.) on synthetic UMI counts where a known true latent correlation must be held fixed across a grid of expression levels.

## When NOT to use

- When the goal is differential co-expression testing on real data rather than estimator benchmarking under known ground truth
- When the underlying generative model must differ from the Poisson measurement process, since the paper's measurement model uses only a Poisson distribution and defers negative-binomial count modeling to future work

## Inputs

- Calibrated negative-binomial marginal parameters (mean and dispersion per gene) and size-factor distribution from a reference scRNA-seq dataset
- Target true latent correlation value (e.g., rho = 0.5)
- Grid of gene expression levels (low/moderate/high)
- Observed raw UMI count matrices for running the competing estimators on identical simulated data

## Outputs

- Synthetic gene-pair UMI count matrices with known true latent correlation
- Per-method bias curves as a function of expression level (Fig. 3a)
- Per-method mean squared error curves (Supplementary Fig. 4)
- Aggregated simulation results table (e.g., simulation_bias_mse.csv)

## How to apply

Calibrate negative-binomial marginal mean/dispersion and the size-factor distribution from a real scRNA-seq reference (the paper uses the postmortem brain dataset of Alzheimer's disease patients and controls). Generate latent bivariate expression via a Gaussian copula with a fixed true correlation (rho = 0.5 in Fig. 3a) over a grid of expression levels, then pass latent expression through a Poisson measurement process with varying sequencing depths to produce observed UMI counts. Run the estimator under test (e.g., CSCORE/CSCORE_IRLS with IRLS estimation and post_process_est) plus competing estimators (Pearson and Spearman on UMI counts, rho-sctransform, rho-analytic PR, locCSN, NoiseRegularization, normalisr) on the same simulated datasets. Compute bias and mean squared error of each method's correlation estimates against the true rho at every expression level, and plot bias/MSE curves. The rationale is that only simulation with known latent correlation can separate mean-correlation bias from genuine co-expression signal.

## Related tools

- **CS-CORE / CSCORE (R package)** (Estimator under evaluation; run as CSCORE/CSCORE_IRLS with IRLS estimation and post_process_est on each simulated dataset to estimate co-expressions and check mean-correlation bias) — https://github.com/ChangSuBiostats/CS-CORE
- **sctransform** (Competing co-expression estimator (rho-sctransform) evaluated on the same simulated UMI counts)
- **locCSN** (Competing co-expression estimator evaluated for bias and MSE on simulated gene-pair counts) — https://github.com/xuranw/locCSN
- **NoiseRegularization** (Competing co-expression estimator evaluated for bias and MSE on simulated gene-pair counts) — https://github.com/RuoyuZhang/NoiseRegularization
- **normalisr** (Competing co-expression estimator evaluated for bias and MSE on simulated gene-pair counts) — https://github.com/lingfeiwang/normalisr

## Evaluation signals

- The CS-CORE bias curve should lie near zero across the expression-level grid, consistent with the reported finding that CS-CORE was not subject to mean-correlation bias
- Competing methods should show biased correlation estimates — most severely underestimating co-expressions for low/moderate expression genes and spuriously increasing with expression levels — if the simulation is set up correctly
- CS-CORE should attain the smallest mean squared errors against true rho = 0.5 among all evaluated methods (cf. Supplementary Fig. 4)
- Estimated correlations should be compared against the known true rho = 0.5 at every expression level; deviation from 0.5 quantifies per-method bias
- All methods must be run on identical simulated datasets so bias/MSE differences are attributable to estimator properties, not data variation

## Limitations

- The paper's measurement model uses only a Poisson distribution; adaptation of CS-CORE to other count distributions such as the negative binomial was left to future work
- Conclusions depend on the negative-binomial/size-factor parameters being calibrated from a specific reference dataset (postmortem brain scRNA-seq of Alzheimer's patients and controls); other calibration sources may shift bias curves
- Results address point estimation (bias and MSE); the paper separately found all existing methods suffer from inflated type-I errors, varying degrees of estimation biases, and reduced power, which this simulation design alone does not fully assess

## Evidence

- [results] In simulations with a true correlation of 0.5, correlation estimates from all other methods were inaccurate, with most severely underestimating co-expressions for low/moderate expression genes and spuriously increasing with expression levels: [evidence span withheld — anchor status: not_found]
- [results] CS-CORE accurately estimated co-expressions and was not subject to mean-correlation bias; CS-CORE also achieved the smallest mean squared errors: [evidence span withheld — anchor status: not_found]
- [discussion] It can be useful to adapt CS-CORE to model xij∣zij using other distributions that model nonnegative integers such as the negative binomial distribution: "It can be useful to adapt CS-CORE to model xij∣zij using other distributions that model nonnegative integers such as the negative binomial distribution and we leave it to future work."
- [intro] CS-CORE models the unobserved true gene expression levels as latent variables, linked to the observed UMI counts through a measurement model: "CS-CORE models the unobserved true gene expression levels as latent variables, linked to the observed UMI counts through a measurement model that accounts for both sequencing depth variations and"
- [intro] we found that all the existing methods, including ρ-sctransform and ρ-analytic PR, suffer from inflated type-I errors, varying degrees of estimation biases, reduced power: "we found that all the existing methods, including ρ-sctransform and ρ-analytic PR, suffer from inflated type-I errors, varying degrees of estimation biases, reduced power in detecting co-expressions"
