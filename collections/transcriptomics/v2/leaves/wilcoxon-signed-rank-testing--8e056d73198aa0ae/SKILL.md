---
name: wilcoxon-signed-rank-testing
description: Use when comparing two methods evaluated on the same set of benchmark
  datasets and you need a paired, non-parametric significance test on per-dataset
  scores (e.g. mean ARIs of scISR 0.571 vs raw 0.504, MAGIC 0.461, scImpute 0.286,
  SAVER 0.423, scScope 0.165, scGNN 0.279).
when_to_use_negative:
- Scores are not paired by dataset (independent samples from different benchmark sets)
  — use an unpaired test instead
- Per-dataset score vectors are incomplete, so scISR and the comparator cannot be
  aligned one-to-one across the 25 datasets
- The claim being tested is a two-sided difference with no directional hypothesis
  — a one-sided test would overstate significance
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scISR
  role: The imputation method whose per-dataset ARI and Purity Index scores are being
    tested against competitors; provides the 'treatment' side of every paired test.
  repo: https://github.com/duct317/scISR
- name: R stats
  role: R statistics package used for the paper's statistical implementation (the
    article notes the stats R package is used in their implementation); supplies wilcox.test
    for the signed-rank tests.
provenance:
  source_task_ids:
  - task_005
  - task_007
  source_papers:
  - doi: 10.1038/s41598-022-06500-4
    title: A novel method for single-cell data imputation using subspace regression
  source_content_hash: sha256:09f0d890c8b0e8c21cd9ab4008b1fe8dc83fa6a5ca6b933c7473034b6bda020b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duct317/scISR
  tools:
  - MAGIC
  - R stats
  - SAVER
  - scISR
  - scImpute
  - scScope
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# wilcoxon-signed-rank-testing

## Summary

One-sided Wilcoxon signed-rank testing on paired per-dataset performance scores (ARI, Purity Index) to statistically validate that scISR's subspace-regression imputation beats competing imputation methods and raw data across the 25 scRNA-seq benchmark datasets.

## When to use

Use when comparing two methods evaluated on the same set of benchmark datasets and you need a paired, non-parametric significance test on per-dataset scores (e.g. mean ARIs of scISR 0.571 vs raw 0.504, MAGIC 0.461, scImpute 0.286, SAVER 0.423, scScope 0.165, scGNN 0.279).

## When NOT to use

- Scores are not paired by dataset (independent samples from different benchmark sets) — use an unpaired test instead
- Per-dataset score vectors are incomplete, so scISR and the comparator cannot be aligned one-to-one across the 25 datasets
- The claim being tested is a two-sided difference with no directional hypothesis — a one-sided test would overstate significance

## Inputs

- Per-dataset performance scores (ARI or Purity Index) for scISR on the 25 real scRNA-seq datasets
- Matching per-dataset scores for each comparator: raw data, MAGIC, scImpute, SAVER, scScope, scGNN (and optionally DrImpute, DCA, DeepImpute, BISCUIT)
- Dataset-to-dataset pairing keys (dataset IDs such as GSE84133, GSE75140, GSE45719, E-MTAB-3321, GSE65525)

## Outputs

- One-sided Wilcoxon signed-rank p-values per comparison (scISR vs each imputation method; scISR vs raw data)
- Aggregated comparison table (e.g. ari_comparison_table.csv / stability_summary.csv) with mean ARIs or mean Purity Index plus p-values

## How to apply

After clustering each of the 25 real scRNA-seq datasets with k-means (using the true number of cell types k) and computing Adjusted Rand Indices for raw data, scISR, and each competitor (MAGIC, scImpute, SAVER, scScope, scGNN), pair the per-dataset scores by dataset and run one-sided Wilcoxon signed-rank tests of scISR against every method. The one-sided alternative asserts scISR's scores are greater; the paper reports p = 9.8e-6 against competing methods and p = 3.2e-5 against raw data for ARI, and p = 0.007 for the ARI-based Purity Index where scISR beats raw data on 15/25 datasets. Pairing by dataset is essential because the same 25 datasets are shared across methods, and the signed-rank test is chosen because per-dataset score differences need not be normally distributed.

## Related tools

- **scISR** (The imputation method whose per-dataset ARI and Purity Index scores are being tested against competitors; provides the 'treatment' side of every paired test.) — https://github.com/duct317/scISR
- **R stats** (R statistics package used for the paper's statistical implementation (the article notes the stats R package is used in their implementation); supplies wilcox.test for the signed-rank tests.)

## Examples

```
wilcox.test(ari_scisr, ari_magic, alternative = "greater", paired = TRUE)
```

## Evaluation signals

- Reproduced p-values match the reported ones: 9.8e-6 for scISR vs competing imputation methods and 3.2e-5 for scISR vs raw data on ARI; p = 0.007 for the Purity Index comparison
- scISR has the highest mean ARI (0.571) over raw (0.504) and all competitors, and is the only method whose average Purity Index exceeds raw data
- Purity Index win count matches 15 of 25 datasets where scISR exceeds raw data
- Comparison is paired: each test uses exactly 25 dataset-matched score pairs; missing datasets must be dropped from both sides consistently
- Output table (ari_comparison_table.csv / stability_summary.csv) contains mean score columns plus a p-value column for every comparator

## Limitations

- The test depends on upstream clustering quality (k-means with true k and ARI computation), so any clustering or preprocessing deviation (gene filtering, log2 rescaling) changes the paired scores and hence p-values
- One-sided tests encode the prior hypothesis that scISR is better; they cannot detect that a competitor outperforms scISR
- With n = 25 paired datasets the exact null distribution is discrete, and results are sensitive to ties in per-dataset ARI values
- Article notes missing version information for the R stats package and the scISR software, limiting exact reproducibility

## Evidence

- [results] we use k-means to cluster the cells using the true number of cell types k as the number of clusters.: "we use k-means to cluster the cells using the true number of cell types k as the number of clusters."
- [discussion] stats R package implementation note: "In our implementation, we adopt the lm function that is available in the stats R package."
- [abstract] 25-dataset evaluation scope: "Our comprehensive evaluation using 25 publicly available scRNA-seq datasets and various simulation scenarios against five state-of-the-art methods"
- [readme] README ARI assessment snippet: "print(paste('ARI of clusters using raw data:', round(adjustedRandIndex(cluster_raw, label),3)))"
