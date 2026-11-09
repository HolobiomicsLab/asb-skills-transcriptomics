---
name: benchmark-method-power-comparison
description: Use when validating a statistical detection method's false discovery
  rate calibration and power against existing methods using simulated signal spiked
  into a null dataset with known gold-standard positives.
when_to_use_negative:
- The analysis goal is applying a single method to real data rather than empirically
  evaluating FDR calibration and power against alternatives on simulated ground truth.
- No gold-standard positive gene set or null simulation can be constructed, so real
  (empirical) FDR cannot be computed.
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2269
tools:
- name: Lamian
  role: 'Method under evaluation: XDE test run with lamian_test, test.type=''variable'',
    testvar=2, permutation-based likelihood ratio test, permuiter=100'
  repo: https://github.com/Winnie09/Lamian
- name: TSCAN
  role: Competing differential pseudotime method run on identical simulated datasets
- name: Slingshot
  role: Competing differential pseudotime method run on identical simulated datasets
- name: Monocle
  role: Competing differential pseudotime method run on identical simulated datasets
- name: tradeSeq
  role: Competing differential trajectory method run on identical simulated datasets
- name: Phenopath
  role: Competing covariate-associated pseudotime method run on identical simulated
    datasets
- name: condiments
  role: Competing covariate-associated pseudotime method run on identical simulated
    datasets
- name: milo
  role: Competing differential abundance method run on identical simulated datasets
- name: DAseq
  role: Competing differential abundance method run on identical simulated datasets
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-42841-y
    title: A statistical framework for differential pseudotime analysis with multiple
      single-cell RNA-seq samples
  source_content_hash: sha256:2ed0965426ba5123858a93b8f52934ddfb0527bf71ac3e9cf9b604df5990f3b2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Winnie09/Lamian
  tools:
  - DAseq
  - Lamian
  - Monocle
  - Phenopath
  - Slingshot
  - TSCAN
  - condiments
  - milo
  - tradeSeq
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# benchmark-method-power-comparison

## Summary

Benchmark a differential-expression/pseudotime method against competitors on spike-in simulations built on a null single-cell dataset, comparing real (empirical) FDR versus reported FDR and sensitivity as a function of real FDR (AUC) to demonstrate conservative FDR control and superior power.

## When to use

Use when validating a statistical detection method's false discovery rate calibration and power against existing methods using simulated signal spiked into a null dataset with known gold-standard positives.

## When NOT to use

- The analysis goal is applying a single method to real data rather than empirically evaluating FDR calibration and power against alternatives on simulated ground truth.
- No gold-standard positive gene set or null simulation can be constructed, so real (empirical) FDR cannot be computed.

## Inputs

- Null multi-sample scRNA-seq simulation with two conditions and multiple biological samples per condition (e.g., null HCA-BM data)
- Gene-by-cell expression matrix
- Cell-to-sample annotation
- Pseudotime vector
- Design matrix with first column as intercept
- Gold-standard gene set (e.g., 1814 genes) with trend-only, mean-only, and trend+mean partitions
- Spike-in signal strength grid (e.g., 0.5–4 across five levels)

## Outputs

- Per-method real FDR vs reported FDR at each signal strength (e.g., lamian_fdr_sensitivity_benchmark.csv)
- Per-method sensitivity as a function of real FDR summarized by AUC
- Significant gene calls per method at reported FDR < 0.05

## How to apply

Start from a multi-sample null simulation (e.g., the null HCA-BM dataset) with two conditions and multiple biological samples per condition, formatted as the target method's inputs: a gene-by-cell expression matrix, cell-to-sample annotation, a pseudotime vector, and a design matrix whose first column is the intercept. Spike differential signals at several strengths (e.g., five levels spanning 0.5–4) into a fixed set of gold-standard genes partitioned into trend-only, mean-only, and trend+mean classes. Run the method under evaluation (e.g., Lamian's XDE test with test.type='variable', testvar=2, and a permutation-based likelihood ratio test with permuiter=100) and all competitors (Monocle, TSCAN, Slingshot, tradeSeq, Phenopath, condiments, milo, DAseq) on the identical simulated datasets. Call significant genes per method at reported FDR < 0.05 and compute real (empirical) FDR against reported FDR at each signal strength, plus sensitivity as a function of real FDR summarized by AUC. Validation requires the evaluated method's real FDR to not exceed its reported FDR (conservative calibration) and the highest sensitivity-realFDR AUC among all compared methods.

## Related tools

- **Lamian** (Method under evaluation: XDE test run with lamian_test, test.type='variable', testvar=2, permutation-based likelihood ratio test, permuiter=100) — https://github.com/Winnie09/Lamian
- **TSCAN** (Competing differential pseudotime method run on identical simulated datasets)
- **Slingshot** (Competing differential pseudotime method run on identical simulated datasets)
- **Monocle** (Competing differential pseudotime method run on identical simulated datasets)
- **tradeSeq** (Competing differential trajectory method run on identical simulated datasets)
- **Phenopath** (Competing covariate-associated pseudotime method run on identical simulated datasets)
- **condiments** (Competing covariate-associated pseudotime method run on identical simulated datasets)
- **milo** (Competing differential abundance method run on identical simulated datasets)
- **DAseq** (Competing differential abundance method run on identical simulated datasets)

## Examples

```
res <- lamian.test(expr, cellanno, pseudotime, design, test.type='variable', testvar=2, permuiter=100, resp.type='XDE')
```

## Evaluation signals

- Evaluated method's real (empirical) FDR must not exceed its reported FDR at every signal strength (conservative calibration).
- Competing methods should be detectable as anti-conservative (real FDR exceeds reported FDR), as observed for Monocle, TSCAN, Slingshot, tradeSeq, Phenopath, condiments, milo, and DAseq.
- Evaluated method's sensitivity-realFDR AUC must be the highest among all compared methods.
- Power (sensitivity) of the evaluated method should increase monotonically with spike-in signal strength.
- Benchmark table (e.g., lamian_fdr_sensitivity_benchmark.csv) must contain realFDR-vs-reportedFDR differences and AUC comparisons for every method and signal level.

## Limitations

- Benchmark evaluates covariate-associated differential expression along pseudotime (XDE) for one method family; other change types (TDE, TCD, XCD) require the supplementary simulations.
- Lamian modules 1 and 2 do not support Monocle2, Monocle3, or Slingshot trajectories, limiting which trajectory inputs can be benchmarked end-to-end.
- Results depend on the null simulation design (HCA-BM, two conditions, spike-in classes); generalization to other tissues or designs is not established by this benchmark.

## Evidence

- [other] Run Lamian's XDE test with lamian_test using test.type='variable', testvar=2, and the permutation-based likelihood ratio test with the default permuiter=100.: [evidence span withheld — anchor status: not_found]
- [other] Spike differential signals at five strengths spanning 0.5-4 into 1814 gold-standard genes partitioned into trend-only, mean-only, and trend+mean classes.: [evidence span withheld — anchor status: not_found]
- [results] In spike-in simulations across all signal strength levels, the real FDR was smaller than the FDR reported by Lamian.pm (conservative FDR control): [evidence span withheld — anchor status: not_found]
- [discussion] To conduct a hypothesis test comparing two models, we use a permutation-based likelihood ratio test.: "To conduct a hypothesis test comparing two models, we use a permutation-based likelihood ratio test."
- [readme] Lamian is able to more appropriately control the false discovery rate (FDR) when analyzing multi-sample data, a property not offered by any other methods.: "Lamian is able to more appropriately control the false discovery rate (FDR) when analyzing multi-sample data, a property not offered by any other methods."
