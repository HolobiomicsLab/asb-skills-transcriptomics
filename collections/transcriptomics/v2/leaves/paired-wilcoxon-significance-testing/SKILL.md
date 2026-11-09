---
name: paired-wilcoxon-significance-testing
description: Use when you have paired per-fold metric values (e.g., AMI, ARI, HOM,
  NMI from 5-fold cross-validation) for two methods on identical folds and need a
  one-sided significance test of ity.
when_to_use_negative:
- The two sets of metric values are not paired on identical evaluation units (e.g.,
  different folds or datasets per method) — the signed-rank test requires one-to-one
  pairing
- You need to compare more than two methods in one omnibus test, or compare distributions
  of raw per-cell scores rather than fold-level aggregated metrics
- The input is raw translated profiles rather than already-computed fold-level AMI/ARI/HOM/NMI
  values — clustering indexing must run first (calculate_cluster_index) before testing
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: scButterfly
  role: Source of the per-fold clustering metrics (AMI, ARI, HOM, NMI) being tested
    against baseline metrics; its five-fold cross-validation design defines the pairing
    structure
  repo: https://github.com/BioX-NKU/scButterfly
- name: Python
  role: Implementation language for the one-sided paired Wilcoxon signed-rank test
    over the n=140 paired evaluations and for writing benchmark_clustering_metrics.csv
- name: BABEL
  role: Baseline translator whose paired per-fold metrics are compared against scButterfly's
    in the one-sided tests
- name: Polarbear
  role: Baseline translator whose paired per-fold metrics are compared against scButterfly's
    in the one-sided tests
- name: JAMIE
  role: Baseline translator whose paired per-fold metrics are compared against scButterfly's
    in the one-sided tests
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-47418-x
    title: 'scButterfly: a versatile single-cell cross-modality translation method
      via dual-aligned variational autoencoders'
  source_content_hash: sha256:f23907554fe3ce32a7d4e1738380e596f32587a5df271af9966ea3436fe4247b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioX-NKU/scButterfly
  tools:
  - BABEL
  - JAMIE
  - Polarbear
  - Python
  - scButterfly
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# paired-wilcoxon-significance-testing

## Summary

Assesses whether a method's paired per-fold benchmark metrics significantly exceed a baseline's using one-sided paired Wilcoxon signed-rank tests. In the scButterfly paper this confirmed scButterfly-B's superiority over BABEL, Polarbear, and JAMIE across 7 datasets × 5 folds × 2 translation directions (n=140 paired evaluations).

## When to use

Use when you have paired per-fold metric values (e.g., AMI, ARI, HOM, NMI from 5-fold cross-validation) for two methods on identical folds and need a one-sided significance test of superiority.

## When NOT to use

- The two sets of metric values are not paired on identical evaluation units (e.g., different folds or datasets per method) — the signed-rank test requires one-to-one pairing
- You need to compare more than two methods in one omnibus test, or compare distributions of raw per-cell scores rather than fold-level aggregated metrics
- The input is raw translated profiles rather than already-computed fold-level AMI/ARI/HOM/NMI values — clustering indexing must run first (calculate_cluster_index) before testing

## Inputs

- Paired per-fold clustering metric table (rows = dataset × fold × direction, columns = AMI, ARI, HOM, NMI) for scButterfly-B from 5-fold cross-validation by cell
- Identically structured per-fold metric table for each baseline method (BABEL, Polarbear, JAMIE) computed on the same folds
- Unit keys (dataset, fold, direction) linking scButterfly values to baseline values for pairing

## Outputs

- Per-comparison test statistics and one-sided p-values for scButterfly-B vs each baseline on each metric and translation direction
- Aggregated significance table written to benchmark_clustering_metrics.csv

## How to apply

First aggregate fold-level clustering metrics (AMI, ARI, HOM, NMI, computed with calculate_cluster.calculate_cluster_index) from the seven paired scRNA-seq/scATAC-seq datasets (BMMC, MB, CL, MCC, MK, MDS, PBMC) × 5 cross-validation folds × 2 translation directions (RNA to ATAC and ATAC to RNA), giving n=140 paired evaluations. For each baseline (BABEL, Polarbear, JAMIE) and each metric, pair the scButterfly-B fold values with the baseline fold values on the same (dataset, fold, direction) unit and compute the paired differences. Run a one-sided Wilcoxon signed-rank test on these differences with the alternative hypothesis that scButterfly-B's metric is greater, which is the non-parametric test the paper used because per-fold metric differences across 140 paired evaluations are not assumed normal. Record the test statistic and p-value per (method pair, metric, direction) and write the aggregated table to benchmark_clustering_metrics.csv. Report a baseline as significantly outperformed when the one-sided p-value is below the significance threshold, consistent with the paper's conclusion that scButterfly-B's advantages were confirmed by one-sided paired Wilcoxon signed-rank tests.

## Related tools

- **scButterfly** (Source of the per-fold clustering metrics (AMI, ARI, HOM, NMI) being tested against baseline metrics; its five-fold cross-validation design defines the pairing structure) — https://github.com/BioX-NKU/scButterfly
- **Python** (Implementation language for the one-sided paired Wilcoxon signed-rank test over the n=140 paired evaluations and for writing benchmark_clustering_metrics.csv)
- **BABEL** (Baseline translator whose paired per-fold metrics are compared against scButterfly's in the one-sided tests)
- **Polarbear** (Baseline translator whose paired per-fold metrics are compared against scButterfly's in the one-sided tests)
- **JAMIE** (Baseline translator whose paired per-fold metrics are compared against scButterfly's in the one-sided tests)

## Examples

```
from scipy.stats import wilcoxon; stat, p = wilcoxon(scB_fold_metrics, baseline_fold_metrics, alternative='greater')
```

## Evaluation signals

- The paired table has n=140 rows (7 datasets × 5 folds × 2 directions) with matching (dataset, fold, direction) keys between scButterfly-B and each baseline
- Every reported test is one-sided with the alternative hypothesis stating scButterfly's metric is greater, not two-sided
- Per-metric mean/median of scButterfly-B values exceeds the corresponding baseline values in both RNA-to-ATAC and ATAC-to-RNA directions, consistent with reported significance
- The output CSV benchmark_clustering_metrics.csv contains one row per (baseline method, metric, direction) with test statistic and p-value
- Significant p-values are only claimed for pairs where the signed-rank statistic aligns with positive median differences; non-significant pairs are retained rather than dropped

## Limitations

- The one-sided test presumes the direction of superiority is stated a priori and cannot detect baselines that significantly outperform scButterfly
- Fold-level metrics within a dataset are not fully independent (overlapping training folds), so p-values rest on the CV pairing approximation used in the paper
- Ties or zero differences among per-fold metric values (common with discrete AMI/ARI) reduce effective sample size and test power

## Evidence

- [results] with significant advantages confirmed by one-sided paired Wilcoxon signed-rank tests.: [evidence span withheld — anchor status: not_found]
- [methods] Run one-sided paired Wilcoxon signed-rank tests over the paired per-fold metric comparisons to assess scButterfly's superiority over each baseline: [evidence span withheld — anchor status: not_found]
- [methods] Cluster the translated profiles and compute AMI, ARI, HOM, and NMI against ground-truth cell labels with calculate_cluster.calculate_cluster_index.: [evidence span withheld — anchor status: not_found]
- [readme] A series of evaluating method also be integrated in this function, you could get these evaluation using parameters:: [evidence span withheld — anchor status: external_doc]
