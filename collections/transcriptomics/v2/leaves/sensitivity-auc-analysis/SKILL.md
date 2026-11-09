---
name: sensitivity-auc-analysis
description: Use when benchmarking statistical methods on spike-in ground-truth simulations
  (e.g., XDE along pseudotime on null HCA-BM data) and you need empirical FDR calibration
  plus a sensitivity-vs-real-FDR AUC to rank methods by power and honesty.
when_to_use_negative:
- Analyzing real datasets with no spike-in gold standard, where empirical FDR cannot
  be computed against known positives
- Benchmarking trajectory/pseudotime inference accuracy itself rather than differential
  expression detection (Lamian modules 1 and 2 do not support Monocle2, Monocle3,
  or slingshot trajectories)
- Single-sample-per-condition designs where methods like condiments assume one sample
  per condition and cross-sample variability cannot be modeled
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Lamian
  role: 'Method under evaluation: XDE detection via permutation-based likelihood ratio
    test (test.type=''variable'', testvar=2, permuiter=100), whose reported FDR is
    compared to empirical FDR'
  repo: https://github.com/Winnie09/Lamian
- name: TSCAN
  role: Competing differential pseudotime method benchmarked on identical simulated
    datasets; also Lamian's default pseudotime constructor
- name: Monocle
  role: Competing differential pseudotime method run on the same spike-in simulations
    for FDR and sensitivity comparison
- name: Slingshot
  role: Competing differential pseudotime method included in the benchmark
- name: tradeSeq
  role: Competing method for detecting gene expression changes along trajectory lineages
- name: Phenopath
  role: Competing condition-differential pseudotime method (linear-along-pseudotime
    assumption)
- name: condiments
  role: Competing condition-differential pseudotime method (one-sample-per-condition
    assumption)
- name: milo
  role: Competing method for cell abundance changes along pseudotime included in the
    benchmark
- name: DAseq
  role: Competing method for differential cell abundance included in the benchmark
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-42841-y
    title: A statistical framework for differential pseudotime analysis with multiple
      single-cell RNA-seq samples
  source_content_hash: sha256:de502f66c6ed9284fdfab3e29634e1449364f00c69c759ebafcbe8ea2d06dcbc
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# sensitivity-auc-analysis

## Summary

Benchmark skill for evaluating differential expression/pseudotime methods on spike-in simulations: compute real (empirical) FDR against reported FDR and summarize power as sensitivity as a function of real FDR (sensitivity-realFDR AUC), used to show Lamian conservatively controls FDR and outperforms Monocle, TSCAN, Slingshot, tradeSeq, Phenopath, condiments, milo, and DAseq.

## When to use

Use when benchmarking statistical methods on spike-in ground-truth simulations (e.g., XDE along pseudotime on null HCA-BM data) and you need empirical FDR calibration plus a sensitivity-vs-real-FDR AUC to rank methods by power and honesty.

## When NOT to use

- Analyzing real datasets with no spike-in gold standard, where empirical FDR cannot be computed against known positives
- Benchmarking trajectory/pseudotime inference accuracy itself rather than differential expression detection (Lamian modules 1 and 2 do not support Monocle2, Monocle3, or slingshot trajectories)
- Single-sample-per-condition designs where methods like condiments assume one sample per condition and cross-sample variability cannot be modeled

## Inputs

- Gene-by-cell expression matrix from a multi-sample null simulation (e.g., HCA-BM derived)
- Cell-to-sample annotation
- Pseudotime vector
- Design matrix whose first column is the intercept
- Gold-standard spike-in gene labels (trend-only, mean-only, trend+mean) with signal strengths 0.5-4
- Per-method significant gene calls at reported FDR < 0.05

## Outputs

- Real-FDR vs reported-FDR comparison per method per signal strength
- Sensitivity as a function of real FDR per method
- Sensitivity-realFDR AUC summary table (e.g., lamian_fdr_sensitivity_benchmark.csv)

## How to apply

Generate a multi-sample null simulation (two conditions, multiple biological samples per condition) and spike differential signals at several strengths spanning 0.5-4 into gold-standard genes partitioned into trend-only, mean-only, and trend+mean classes. Run the method under evaluation (e.g., Lamian's XDE test via lamian_test with test.type='variable', testvar=2, and the permutation-based likelihood ratio test with permuiter=100) alongside all competitors on identical simulated datasets. Call significant genes per method at reported FDR < 0.05 and compute the real (empirical) FDR against the reported FDR at each signal strength. Compute sensitivity (recall against the spike-in gold standard) as a function of real FDR and summarize the power comparison by the area under this curve (AUC). The method is correctly calibrated if real FDR does not exceed reported FDR, and most powerful if its sensitivity-realFDR AUC is the highest among all compared methods.

## Related tools

- **Lamian** (Method under evaluation: XDE detection via permutation-based likelihood ratio test (test.type='variable', testvar=2, permuiter=100), whose reported FDR is compared to empirical FDR) — https://github.com/Winnie09/Lamian
- **TSCAN** (Competing differential pseudotime method benchmarked on identical simulated datasets; also Lamian's default pseudotime constructor)
- **Monocle** (Competing differential pseudotime method run on the same spike-in simulations for FDR and sensitivity comparison)
- **Slingshot** (Competing differential pseudotime method included in the benchmark)
- **tradeSeq** (Competing method for detecting gene expression changes along trajectory lineages)
- **Phenopath** (Competing condition-differential pseudotime method (linear-along-pseudotime assumption))
- **condiments** (Competing condition-differential pseudotime method (one-sample-per-condition assumption))
- **milo** (Competing method for cell abundance changes along pseudotime included in the benchmark)
- **DAseq** (Competing method for differential cell abundance included in the benchmark)

## Examples

```
res <- lamian.test(test.type = 'variable', testvar = 2, permuiter = 100)
```

## Evaluation signals

- Lamian's real FDR must not exceed its reported FDR at every signal strength (conservative FDR calibration)
- All competing methods should underestimate real FDR relative to reported FDR on these simulations
- Lamian's sensitivity-realFDR AUC must be the highest among all compared methods
- Lamian's power should increase monotonically with spike-in signal strength across the 0.5-4 range
- Output CSV contains realFDR-vs-reportedFDR difference and AUC rows for every method and signal level

## Limitations

- Simulation evaluation for TDE, TCD, and XCD detection is deferred to Supplementary Notes rather than fully specified in the main text
- Existing methods' power comparisons are confounded by their differing assumptions (e.g., Phenopath assumes linear changes along pseudotime; condiments assumes one sample per condition)
- A systematic comparison of sample-harmonization methods was explicitly beyond the scope of the study

## Evidence

- [abstract] Unlike existing methods that ignore sample variability, Lamian draws statistical inference after accounting for cross-sample variability and hence substantially reduces sample-specific false: "Unlike existing methods that ignore sample variability, Lamian draws statistical inference after accounting for cross-sample variability and hence substantially reduces sample-specific false"
- [methods] Call significant genes per method at reported FDR < 0.05 and compute real (empirical) FDR against reported FDR at each signal strength, plus sensitivity as a function of real FDR summarized by AUC.: [evidence span withheld — anchor status: not_found]
- [discussion] To conduct a hypothesis test comparing two models, we use a permutation-based likelihood ratio test.: "To conduct a hypothesis test comparing two models, we use a permutation-based likelihood ratio test."
- [readme] As a result, Lamian is able to more appropriately control the false discovery rate (FDR) when analyzing multi-sample data, a property not offered by any other methods.: "As a result, Lamian is able to more appropriately control the false discovery rate (FDR) when analyzing multi-sample data, a property not offered by any other methods."
