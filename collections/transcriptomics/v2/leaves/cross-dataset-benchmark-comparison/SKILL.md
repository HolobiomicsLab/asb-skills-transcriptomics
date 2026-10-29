---
name: cross-dataset-benchmark-comparison
description: Use when comparing competing feature sets or representations for cell
  clustering across several scRNA-seq datasets, requiring replicate-level metrics
  (AMI) against known subpopulation labels to show consistency.
when_to_use_negative:
- Cells have no known subpopulation labels, so AMI against ground truth cannot be
  computed
- The goal is a single-dataset clustering analysis rather than a controlled feature-type
  comparison across datasets
- Only raw SNV/GE matrices exist and eeSNV features cannot yet be derived via SSrGE
  fitting
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: SSrGE
  role: Fits sparse LASSO models linking SNV matrix (X) to GE matrix (Y), ranks eeSNVs,
    and transforms cells into reduced eeSNV feature space for benchmarking
  repo: https://github.com/lanagarmire/SSrGE.git
- name: scikit-learn 0.18
  role: Provides PCA, factor analysis, K-means, and agglomerative clustering with
    WARD linkage for the five clustering strategies
- name: Numpy
  role: Array operations for building and resampling SNV/GE matrices across bootstrap
    runs
- name: Scipy
  role: Numerical support library in the SSrGE environment used for the benchmark
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-018-07170-5
    title: Using single nucleotide variations in single-cell RNA-seq to identify subpopulations
      and genotype-phenotype linkage
  source_content_hash: sha256:2cd724f82bc48fb82543719f841db2e8bc81afe5bb9bd97e176fdff6445fa800
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/SSrGE.git
  tools:
  - Numpy
  - Python
  - SSrGE
  - Scipy
  - scikit-learn
  - scikit-learn 0.18
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# cross-dataset-benchmark-comparison

## Summary

Benchmark two feature representations (eeSNVs derived by SSrGE vs full gene expression matrices) for scRNA-seq subpopulation identification across multiple GEO datasets and clustering algorithms, quantifying performance with adjusted mutual information (AMI) over 30 bootstrap runs.

## When to use

Use when comparing competing feature sets or representations for cell clustering across several scRNA-seq datasets, requiring replicate-level metrics (AMI) against known subpopulation labels to show consistency.

## When NOT to use

- Cells have no known subpopulation labels, so AMI against ground truth cannot be computed
- The goal is a single-dataset clustering analysis rather than a controlled feature-type comparison across datasets
- Only raw SNV/GE matrices exist and eeSNV features cannot yet be derived via SSrGE fitting

## Inputs

- Five GEO scRNA-seq datasets (GSE73121, GSE51372, GSE67980, GSE57872, GSE75688)
- Per-dataset SNV matrix (n_cells x n_SNVs, binary)
- Per-dataset gene expression matrix (n_cells x n_Genes)
- Known subpopulation labels per cell

## Outputs

- Reduced eeSNV feature matrices (from ssrge.transform(X))
- Per-run AMI scores for each dataset x feature type x clustering strategy over 30 bootstrap runs
- Aggregated comparison table (e.g., ami_comparison_results.csv) with mean and variability of AMI

## How to apply

For each dataset (e.g., GSE73121, GSE51372, GSE67980, GSE57872, GSE75688), build an n_cells x n_SNVs binary SNV matrix and an n_cells x n_Genes GE matrix, then fit SSrGE with ssrge.fit(X, Y) and derive eeSNV features via ssrge.rank_eeSNVs() and ssrge.transform(X). Run five clustering strategies on both feature types using scikit-learn 0.18 (Python 2.7): PCA or factor-analysis reduction followed by K-means, PCA/FA followed by agglomerative clustering with WARD linkage, and SIMLR. Repeat each clustering over 30 bootstrap resampling runs per dataset per feature type, computing AMI against known subpopulation labels each run. Aggregate mean and variability of AMI per dataset, feature type, and clustering strategy (e.g., into ami_comparison_results.csv), and check the aggregated pattern reproduces the expected figure (eeSNV above GE in all datasets, AMI bounded in [0, 1]).

## Related tools

- **SSrGE** (Fits sparse LASSO models linking SNV matrix (X) to GE matrix (Y), ranks eeSNVs, and transforms cells into reduced eeSNV feature space for benchmarking) — https://github.com/lanagarmire/SSrGE.git
- **scikit-learn 0.18** (Provides PCA, factor analysis, K-means, and agglomerative clustering with WARD linkage for the five clustering strategies)
- **Numpy** (Array operations for building and resampling SNV/GE matrices across bootstrap runs)
- **Scipy** (Numerical support library in the SSrGE environment used for the benchmark)

## Examples

```
from garmire_SSrGE.ssrge import SSrGE; ssrge = SSrGE(); ssrge.fit(X, Y); ranked_feature = ssrge.rank_eeSNVs(); X_r = ssrge.transform(X)
```

## Evaluation signals

- Aggregated AMI values reproduce the Fig. 2 pattern: eeSNV features achieve higher accuracy than GE for identifying subpopulations in all datasets tested
- All AMI scores fall within the valid range [0, 1]
- Each dataset x feature type x clustering strategy cell is backed by 30 independent bootstrap runs, with reported mean and variability
- Reduced eeSNV matrix shapes returned by ssrge.transform(X) are consistent with the number of selected eeSNVs (X_r.shape < X.shape)
- The comparison is robust across all five clustering approaches (PCA/FA + K-means or agglo-WARD, and SIMLR)

## Limitations

- SSrGE may miss some driver mutations due to the incomplete DNA coverage of scRNA-seq reads
- In case of correlated features, LASSO may arbitrarily select one of the highly correlated SNV features corresponding to GE
- Explanations for the eeSNV accuracy advantage are speculative and not experimentally verified

## Evidence

- [abstract] eeSNV vs GE benchmark consistency: "In all the datasets tested, eeSNVs achieve better accuracies than gene expression for identifying subpopulations."
- [methods] ranking eeSNVs: "ranked_feature = ssrge.rank_eeSNVs()"
- [discussion] LASSO correlation limitation: "but in case of correlated features, LASSO may select one of those highly correlated SNV features that correspond to GE"
- [readme] SSrGE as dimension reduction/feature selection: "This procedure can be used as  a dimension reduction/feature selection procedure or as a feature ranking. It is based on the Scikit-Learn library and is easy to re-implement."
- [readme] input matrix specification: "SSrGE takes as input two matrices (A SNV matrix (n_cells x n_SNVs) and a Gene matrix (n_cells x n_Genes)"
