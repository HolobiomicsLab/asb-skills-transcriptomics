---
name: single-cell-rna-seq-simulation
description: Use when a new differential expression / differential pseudotime method
  must be evaluated for FDR control and power against gold-standard spike-in signals
  on multi-sample scRNA-seq data with pseudotime and a condition design matrix.
when_to_use_negative:
- The goal is evaluating trajectory/pseudotime inference uncertainty or topology rather
  than covariate-associated differential expression (XDE) detection
- Data lack multiple biological samples per condition — existing methods that ignore
  sample-to-sample variation would not be meaningfully benchmarked
- Simulating non-expression modalities (e.g. single-cell ATAC-seq) is needed; Lamian's
  model extension to other data types is deferred to future work
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: Lamian
  role: XDE differential expression testing via lamian_test with test.type='variable',
    testvar=2, permutation-based likelihood ratio test (permuiter=100); supplies reported
    FDR to compare against empirical FDR
  repo: https://github.com/Winnie09/Lamian
- name: TSCAN
  role: Competing method in the benchmark and default pseudotime construction within
    Lamian
  repo: https://github.com/Winnie09/Lamian
- name: Monocle
  role: Competing differential pseudotime method run on identical simulated datasets
- name: Slingshot
  role: Competing differential pseudotime method run on identical simulated datasets
- name: tradeSeq
  role: Competing method for changes in trajectory lineages
- name: Phenopath
  role: Competing pseudotime method identifying changes across conditions
- name: condiments
  role: Competing pseudotime method identifying changes across conditions
- name: milo
  role: Competing method for changes in cell abundance along pseudotime
- name: DAseq
  role: Competing method for changes in cell abundance along pseudotime
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-42841-y
    title: A statistical framework for differential pseudotime analysis with multiple
      single-cell RNA-seq samples
  source_content_hash: sha256:e481e6c7b2b20997850c19cfd29720cbcbf7399673e121d79525399fdd0af5ee
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
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# single-cell-rna-seq-simulation

## Summary

Generate spike-in multi-sample single-cell RNA-seq simulations on a null dataset (HCA bone marrow) to benchmark FDR control and statistical power of differential expression methods along pseudotime, using gold-standard genes spiked at graded signal strengths (0.5–4).

## When to use

Use when a new differential expression / differential pseudotime method must be evaluated for FDR control and power against gold-standard spike-in signals on multi-sample scRNA-seq data with pseudotime and a condition design matrix.

## When NOT to use

- The goal is evaluating trajectory/pseudotime inference uncertainty or topology rather than covariate-associated differential expression (XDE) detection
- Data lack multiple biological samples per condition — existing methods that ignore sample-to-sample variation would not be meaningfully benchmarked
- Simulating non-expression modalities (e.g. single-cell ATAC-seq) is needed; Lamian's model extension to other data types is deferred to future work

## Inputs

- Gene-by-cell expression matrix from a null multi-sample scRNA-seq dataset (e.g. HCA bone marrow hca_bm_saver)
- Cell-to-sample annotation (hca_bm_cellanno)
- Low-dimensional representation / PCA (hca_bm_pca)
- Pseudotime vector per cell
- Design matrix with first column the intercept and covariate(s) of interest (e.g. condition as testvar=2)
- Gold-standard gene list (1814 genes: trend-only, mean-only, trend+mean classes)

## Outputs

- lamian_fdr_sensitivity_benchmark.csv containing realFDR-vs-reportedFDR difference and sensitivity-realFDR AUC for all methods
- Per-method significant gene calls at reported FDR < 0.05 at each of five spike-in signal strengths
- Empirical (real) FDR estimates per signal strength per method

## How to apply

Build a multi-sample null simulation with two conditions and multiple biological samples per condition, formatted as the four Lamian inputs: a gene-by-cell expression matrix, cell-to-sample annotation, a pseudotime vector, and a design matrix whose first column is the intercept (HCA-BM hca_bm_saver / hca_bm_pca / hca_bm_cellanno example data structures). Spike differential signals at five strengths spanning 0.5–4 into 1814 gold-standard genes partitioned into trend-only, mean-only, and trend+mean classes. Run Lamian's XDE test with lamian_test using test.type='variable', testvar=2, and the permutation-based likelihood ratio test with the default permuiter=100; run competing methods (Monocle, TSCAN, Slingshot, tradeSeq, Phenopath, condiments, milo, DAseq) on identical simulated datasets. Call significant genes at reported FDR < 0.05 and compute real (empirical) FDR against reported FDR at each signal strength, plus sensitivity as a function of real FDR summarized by AUC.

## Related tools

- **Lamian** (XDE differential expression testing via lamian_test with test.type='variable', testvar=2, permutation-based likelihood ratio test (permuiter=100); supplies reported FDR to compare against empirical FDR) — https://github.com/Winnie09/Lamian
- **TSCAN** (Competing method in the benchmark and default pseudotime construction within Lamian) — https://github.com/Winnie09/Lamian
- **Monocle** (Competing differential pseudotime method run on identical simulated datasets)
- **Slingshot** (Competing differential pseudotime method run on identical simulated datasets)
- **tradeSeq** (Competing method for changes in trajectory lineages)
- **Phenopath** (Competing pseudotime method identifying changes across conditions)
- **condiments** (Competing pseudotime method identifying changes across conditions)
- **milo** (Competing method for changes in cell abundance along pseudotime)
- **DAseq** (Competing method for changes in cell abundance along pseudotime)

## Examples

```
res <- lamian.test(expr=expr_matrix, cellanno=cellanno, pseudotime=pseudotime, design=design, test.type='variable', testvar=2, permuiter=100)
```

## Evaluation signals

- Lamian's real FDR must not exceed its reported FDR at every signal strength (conservative calibration)
- All competing methods must be checked for underestimation of real FDR relative to reported FDR
- Lamian's sensitivity-vs-realFDR AUC must be the highest among all compared methods
- Lamian's power should increase monotonically with spike-in signal strength (0.5–4)
- Output CSV must contain realFDR-vs-reportedFDR difference and sensitivity-realFDR AUC rows for all nine methods

## Limitations

- A systematic comparison of harmonization methods used upstream of simulation is beyond the study's scope
- Simulations for TDE, TCD, and XCD detection are deferred to Supplementary Notes; this skill targets the XDE spike-in benchmark
- Existing methods underestimate real FDR because they ignore cross-sample variability, so their reported FDR values cannot be trusted at face value

## Evidence

- [discussion] To conduct a hypothesis test comparing two models, we use a permutation-based likelihood ratio test.: "To conduct a hypothesis test comparing two models, we use a permutation-based likelihood ratio test."
- [abstract] Lamian accounts for cross-sample variability, reducing sample-specific false discoveries not generalizable to new samples: "Unlike existing methods that ignore sample variability, Lamian draws statistical inference after accounting for cross-sample variability and hence substantially reduces sample-specific false"
- [methods] hca_bm example data structures used as simulation substrate: "Here, we use example data **hca_bm_saver**, **hca_bm_pca**, **hca_bm_cellanno** to demonstrate their data structures."
- [readme] FDR control rationale from README: "Lamian is able to more appropriately control the false discovery rate (FDR) when analyzing multi-sample data, a property not offered by any other methods."
