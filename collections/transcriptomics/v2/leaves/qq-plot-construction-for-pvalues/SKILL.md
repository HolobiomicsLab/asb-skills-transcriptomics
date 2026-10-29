---
name: qq-plot-construction-for-pvalues
description: Use when you have a large set of null (permutation/simulated) association
  p-values and need to assess statistical calibration — e.g., verifying type I error
  control against identity-line expectations in the tails.
when_to_use_negative:
- When p-values come from real (non-null) GWAS/TWAS data with true associations —
  Q-Q deviation then confounds polygenic signal with inflation, and calibration cannot
  be assessed
- When only a handful of p-values are available — tail calibration at 10^-4 to 2.5x10^-6
  requires ~10^6 null replicates to be meaningful
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: OTTERS
  role: Source of null ACAT-O omnibus TWAS p-values via testing.py with Stage I-trained
    eQTL weights
  repo: https://github.com/daiqile96/OTTERS
- name: numpy
  role: Simulate null phenotypes from N(0,1) and compute sorted observed quantiles
- name: scipy
  role: Compute expected uniform quantiles for the Q-Q plot and empirical type I error
    rates
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-36862-w
    title: 'OTTERS: a powerful TWAS framework leveraging summary-level reference data'
  source_content_hash: sha256:08aee16a77754777c526ec336f65437db96dab518081aafc9f7e22a16ecd006b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/daiqile96/OTTERS
  tools:
  - OTTERS
  - Python
  - numpy
  - scipy
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# qq-plot-construction-for-pvalues

## Summary

Construct a quantile-quantile (Q-Q) plot of observed vs expected TWAS p-values under the null to visually verify calibration of the OTTERS omnibus (ACAT-O) test, complementing the empirical type I error table.

## When to use

Use when you have a large set of null (permutation/simulated) association p-values and need to assess statistical calibration — e.g., verifying type I error control against identity-line expectations in the tails.

## When NOT to use

- When p-values come from real (non-null) GWAS/TWAS data with true associations — Q-Q deviation then confounds polygenic signal with inflation, and calibration cannot be assessed
- When only a handful of p-values are available — tail calibration at 10^-4 to 2.5x10^-6 requires ~10^6 null replicates to be meaningful

## Inputs

- Null TWAS p-values (ACAT-O omnibus p-values from simulated null GWAS replicates)
- Simulated null phenotypes from N(0,1) (2000 phenotypes, p_causal=0.001, h_e2=0.1)
- Empirical type I error rates per p-value threshold (10^-2, 10^-4, 2.5x10^-6)

## Outputs

- qq_plot.png — Q-Q plot of observed vs expected null TWAS p-values
- type1_error_table.csv — per-threshold empirical type I error rates

## How to apply

Generate null TWAS replicates (e.g., 10^6 replicates from 2000 quantitative phenotypes drawn from N(0,1) with p_causal=0.001 and h_e2=0.1) by running OTTERS testing.py with Stage I-trained eQTL weights across P0.001, P0.05, lassosum, SDPR, and PRScs models, collecting the ACAT-O omnibus p-values. Sort the null p-values and pair them against expected uniform quantiles (expected p_i = i/(N+1) or scipy.stats.probplot with a uniform distribution), typically on -log10 axes so tail deviation at 10^-4 to 2.5x10^-6 is visible. Plot observed vs expected quantiles with matplotlib, add the identity line, and save to qq_plot.png alongside a type1_error_table.csv of empirical type I error rates at thresholds 10^-2, 10^-4, and 2.5x10^-6.

## Related tools

- **OTTERS** (Source of null ACAT-O omnibus TWAS p-values via testing.py with Stage I-trained eQTL weights) — https://github.com/daiqile96/OTTERS
- **numpy** (Simulate null phenotypes from N(0,1) and compute sorted observed quantiles)
- **scipy** (Compute expected uniform quantiles for the Q-Q plot and empirical type I error rates)

## Examples

```
import numpy as np, scipy.stats as st; obs = np.sort(null_pvals); exp = (np.arange(len(obs))+1)/(len(obs)+1); plt.plot(-np.log10(exp), -np.log10(obs), '.'); plt.plot([0, max(-np.log10(exp))]*2)
```

## Evaluation signals

- Q-Q plot tracks the identity line, matching Supplementary Fig. S1 of the OTTERS paper
- Empirical type I error rates are well controlled at thresholds 10^-4 to 2.5x10^-6, consistent with Supplementary Table S1
- Only slight inflation is expected at the loosest threshold 10^-2; larger deviations indicate mis-calibration or simulation error
- Total replicate count should reach ~10^6 null TWAS p-values (2000 simulated phenotypes) so extreme tail quantiles are populated
- type1_error_table.csv covers all three thresholds (10^-2, 10^-4, 2.5x10^-6) with rates near nominal values

## Limitations

- Q-Q plots alone do not quantify type I error; they must be paired with per-threshold empirical rate tables
- Calibration was assessed with eQTLGen (European-ancestry, n=31,684 whole blood) summary reference data; transferability to other ancestries is limited since available eQTL summary statistics are mainly from European-descent individuals

## Evidence

- [supplementary] Calibration under null simulations: [evidence span withheld — anchor status: not_found]
- [abstract] Simulation-based validation rationale: "We show that OTTERS is a practical and powerful TWAS tool by both simulations and application studies."
- [intro] Omnibus test source of null p-values: "OTTERS first constructs individual TWAS tests and p values using eQTL weights trained by each of the PRS techniques outlined above, and then calculates an omnibus test p value"
