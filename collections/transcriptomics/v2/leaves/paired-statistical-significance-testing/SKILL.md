---
name: paired-statistical-significance-testing
description: Use when two or more methods have been evaluated on the same set of datasets
  and a per-dataset paired comparison of scores (e.g., AMI/NMI/ARI) must establish
  statistically significant ity rather than just a higher mean rank.
when_to_use_negative:
- Scores are not paired per dataset (methods evaluated on disjoint dataset sets) —
  a paired t-test is then inappropriate
- Only a single dataset is available, leaving n = 1 pairs with no across-dataset replication
  to test
- The goal is per-dataset rank ordering rather than aggregate significance across
  datasets
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: scikit-learn
  role: Computes the external-validation metrics (AMI, NMI, ARI) against ground-truth
    cell-type labels that feed the paired t-tests
- name: NumPy
  role: Numerical aggregation of per-dataset score differences used in the paired
    t-tests
- name: scMDC
  role: Focal method whose per-dataset scores are compared against competitors via
    one-sided paired t-tests
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-35031-9
    title: Clustering of single-cell multi-omics data with a multimodal deep learning
      method
  source_content_hash: sha256:e0a01bd8df6952e90e6a7483793add28f0d1f4f91f798607a6e5a88fd60f5ce0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BREM-SC
  - CiteFuse
  - Cobolt
  - IDEC
  - NumPy
  - SC3
  - SCVIS
  - SKlearn
  - Scanpy
  - Seurat V4
  - Specter
  - TotalVI
  - Tscan
  - h5py
  - scMDC
  - scMM
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.65
n_claims: 20
---

# Paired statistical significance testing

## Summary

Apply one-sided paired t-tests across a fixed set of datasets to test whether a focal method's clustering scores (AMI, NMI, ARI) exceed each competitor's, exactly as scMDC's benchmark established significance over ten competing methods.

## When to use

Use when two or more methods have been evaluated on the same set of datasets and a per-dataset paired comparison of scores (e.g., AMI/NMI/ARI) must establish statistically significant superiority rather than just a higher mean rank.

## When NOT to use

- Scores are not paired per dataset (methods evaluated on disjoint dataset sets) — a paired t-test is then inappropriate
- Only a single dataset is available, leaving n = 1 pairs with no across-dataset replication to test
- The goal is per-dataset rank ordering rather than aggregate significance across datasets

## Inputs

- Per-dataset clustering scores (AMI, NMI, ARI) for the focal method and each competitor, computed against ground-truth cell-type labels Y from the h5 input files (X1 = RNA, X2 = ADT/ATAC)
- Ranked score tables per dataset (e.g., results_table.csv, smage_seq_benchmark_results.csv, supp_table4_ttest_results.csv)

## Outputs

- One-sided paired t-test p-values per focal-vs-competitor comparison across the shared dataset set
- CSV of scores, ranks, and p-values (e.g., supp_table4_ttest_results.csv) reproducing the paper's significance tables

## How to apply

For each competitor, compute the per-dataset score differences (focal method minus competitor) over the shared dataset collection (nine CITE-seq datasets for the CITE-seq benchmark; thirteen CITE-seq plus SMAGE-seq datasets for ablations; four SMAGE-seq datasets including multi-batch PBMC13K), then run a one-sided paired t-test of the differences. Metrics are computed with scikit-learn external-validation metrics (AMI, NMI, ARI) against ground-truth cell-type labels before testing. Interpret p-values against the paper's pattern: significance for scMDC over all ten CITE-seq competitors, and note that small n (e.g., n = 4 SMAGE-seq datasets) can yield non-significant results like the scMDC-ATAC comparison (P-value = 0.07). Report the p-values alongside per-dataset scores and ranks (e.g., results_table.csv, supp_table4_ttest_results.csv).

## Related tools

- **scikit-learn** (Computes the external-validation metrics (AMI, NMI, ARI) against ground-truth cell-type labels that feed the paired t-tests)
- **NumPy** (Numerical aggregation of per-dataset score differences used in the paired t-tests)
- **scMDC** (Focal method whose per-dataset scores are compared against competitors via one-sided paired t-tests)

## Examples

```
from sklearn.metrics import adjusted_mutual_info_score, normalized_mutual_info_score, adjusted_rand_score; ari = adjusted_rand_score(y_true, y_pred)
```

## Evaluation signals

- Every focal-vs-competitor pair yields a p-value reproduced in a CSV (e.g., supp_table4_ttest_results.csv), covering all competitors on all shared datasets
- The significance pattern matches the paper: significant one-sided t-test improvements of scMDC over all ten CITE-seq competitors on AMI, NMI, and ARI
- Small-sample exceptions reproduce, e.g., the scMDC vs. scMDC-ATAC comparison on the n = 4 SMAGE-seq datasets showing P-value = 0.07
- Per-method ranks across datasets are consistent with the t-test conclusions (scMDC rank 1 across AMI, NMI, ARI)
- The number of paired observations equals the number of shared datasets (nine CITE-seq, four SMAGE-seq, or thirteen combined)

## Limitations

- Small sample sizes undermine power: with only four SMAGE-seq datasets, the scMDC vs. scMDC-ATAC comparison was not significant (P-value = 0.07)
- Tests depend on identical preprocessing (h5 files with X1/X2/Y, Scanpy normalization of mRNA and ADT/ATAC separately) so that scores are truly paired

## Evidence

- [results] one-sided paired t-tests confirm its improvements over all competing methods are statistically significant: [evidence span withheld — anchor status: not_found]
- [results] scMDC-ATAC n=4 exception: [evidence span withheld — anchor status: not_found]
- [methods] metric computation with scikit-learn: [evidence span withheld — anchor status: not_found]
- [methods] paired t-test step in benchmark workflow: [evidence span withheld — anchor status: not_found]
