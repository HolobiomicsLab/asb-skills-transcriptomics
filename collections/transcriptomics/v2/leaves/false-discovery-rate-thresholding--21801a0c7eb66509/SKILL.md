---
name: false-discovery-rate-thresholding
description: Use when you have a fitted scCODA CAResult (HMC posterior samples of
  spike-and-slab indicators) and must call credible cell-type effects while controlling
  the expected false discovery rate at a chosen level.
when_to_use_negative:
- Input is not a scCODA Bayesian posterior — e.g., p-values from ANCOM, ANCOM-BC,
  t tests, or Poisson/Beta-Binomial regression, where the paper shows nominal FDR
  control fails and compositionality is ignored.
- The analysis goal requires interpreting effects in absolute abundance terms rather
  than relative to a reference cell type.
- Response heterogeneity between samples within a condition is of interest — scCODA
  does not model within-condition response variability, so FDR calls would miss donor-specific
  effects.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: scCODA
  role: Provides credible_effects()/set_fdr on the CAResult object for direct-posterior-probability
    FDR threshold determination of spike-and-slab effect calls
  repo: https://github.com/theislab/scCODA
- name: TensorFlow / tensorflow-probability
  role: Backend for sample_hmc() that draws the posterior samples of spike-and-slab
    indicators from which posterior inclusion probabilities are computed before thresholding
provenance:
  source_task_ids:
  - task_003
  - task_001
  - task_002
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-27150-6
    title: scCODA is a Bayesian model for compositional single-cell data analysis
  source_content_hash: sha256:b700ebcf850c42dde84d3c9192a56cba7c3908c571422998afe624c7b6c5295a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/scCODA
  tools:
  - ALDEx2
  - ANCOM
  - ANCOM-BC
  - Python
  - Python (>=3.8)
  - TensorFlow
  - TensorFlow / tensorflow-probability
  - arviz
  - scCODA
  - tensorflow-probability
  - tensorflow-probability (>=0.12)
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# false-discovery-rate-thresholding

## Summary

Calls statistically credible cell-type effects by thresholding scCODA's spike-and-slab posterior inclusion probabilities via direct-posterior-probability FDR determination (credible_effects()/set_fdr), at target levels such as 0.05 or 0.2.

## When to use

Use when you have a fitted scCODA CAResult (HMC posterior samples of spike-and-slab indicators) and must call credible cell-type effects while controlling the expected false discovery rate at a chosen level.

## When NOT to use

- Input is not a scCODA Bayesian posterior — e.g., p-values from ANCOM, ANCOM-BC, t tests, or Poisson/Beta-Binomial regression, where the paper shows nominal FDR control fails and compositionality is ignored.
- The analysis goal requires interpreting effects in absolute abundance terms rather than relative to a reference cell type.
- Response heterogeneity between samples within a condition is of interest — scCODA does not model within-condition response variability, so FDR calls would miss donor-specific effects.

## Inputs

- CAResult object from sccoda.util.comp_ana.CompositionalAnalysis after sample_hmc() inference (posterior samples of intercepts and spike-and-slab effect indicators)
- Target FDR level (e.g., 0.05, 0.1, or 0.2)
- Pre-specified or automatically selected reference cell type

## Outputs

- Effects data frame with a populated 'Final Parameter' column (0 = not credible, nonzero = credible)
- Credible-effects set at the chosen FDR level (e.g., from credible_effects())
- Benchmark metrics (TPR/FDR per setting) when applied across synthetic replicate grids

## How to apply

After fitting the hierarchical Dirichlet-Multinomial model with Logit-normal spike-and-slab priors via sample_hmc(), compute the posterior inclusion probability of each cell-type-by-covariate effect from the spike-and-slab indicators. Apply direct-posterior-probability FDR threshold determination by calling set_fdr(target_fdr_level=0.2) or credible_effects() directly at the desired FDR level (the paper uses 0.05 for benchmarking and 0.2 for power analysis and the Alzheimer's GSE98969 analysis). Effects must be interpreted relative to the reference cell type, whose effect is constrained to zero. Read the 'Final Parameter' column of the effects data frame: 0 flags non-credible effects, nonzero flags credible effects. Rationale: standard univariate tests ignore compositional negative correlations and the paper shows competing microbiome methods (ANCOM, ANCOM-BC) could not adequately control the FDR, whereas scCODA's Bayesian thresholding reliably does.

## Related tools

- **scCODA** (Provides credible_effects()/set_fdr on the CAResult object for direct-posterior-probability FDR threshold determination of spike-and-slab effect calls) — https://github.com/theislab/scCODA
- **TensorFlow / tensorflow-probability** (Backend for sample_hmc() that draws the posterior samples of spike-and-slab indicators from which posterior inclusion probabilities are computed before thresholding)

## Examples

```
from sccoda.util.comp_ana import CompositionalAnalysis; res = CompositionalAnalysis(dat, formula="condition", reference_cell_type="microglia 1").run_hmc(); res.set_fdr(target_fdr_level=0.2); print(res.credible_effects())
```

## Evaluation signals

- The 'Final Parameter' column of the effects data frame is fully populated, with 0 marking non-credible and nonzero marking credible effects.
- At nominal FDR 0.05 on the synthetic benchmark, false discovery rate stays at or near the nominal level while true positive rate remains high (scCODA MCC 0.64, AUC 0.99, average precision 0.94).
- Credible-effect calls at FDR 0.2 reproduce Supplementary Data 2 for GSE98969: credible DAM increase in cortex, no credible change in cerebellum relative to microglia 1.
- Effects of the reference cell type are zero and all reported effects are interpretable only relative to that reference.
- Power-analysis inversion of the detection results (quasibinomial GLM of TPR on sample size at FDR 0.2) reaches fit quality R² ≈ 0.937.

## Limitations

- Credible changes are only interpretable relative to the selected reference cell type; changing the reference changes which effects are called credible.
- The threshold acts on posterior inclusion probabilities from a spike-and-slab prior, not on p-values, so it is not interchangeable with Benjamini-Hochberg corrections applied to frequentist test statistics.
- Rival methods at nominal FDR 0.05 (ANCOM, ANCOM-BC) failed to control the actual false discovery rate, so per-method calibration must be re-checked when comparing calls across tools.

## Evidence

- [abstract] scCODA demonstrated excellent detection performance, while reliably controlling for false discoveries: "scCODA demonstrated excellent detection performance, while reliably controlling for false discoveries"
- [methods] The ``Final Parameter`` column of the effects data frame shows the significances.: "The ``Final Parameter`` column of the effects data frame shows the significances."
- [discussion] Other methods from the field of microbiome data analysis, such as ANCOM and ANCOM-BC, showed similar detection power, but could not adequately control the false discovery rate: "such as ANCOM and ANCOM-BC, showed similar detection power, but could not adequately control the false discovery rate"
- [readme] scCODA allows for identification of compositional changes in high-throughput sequencing count data: "scCODA allows for identification of compositional changes in high-throughput sequencing count data"
