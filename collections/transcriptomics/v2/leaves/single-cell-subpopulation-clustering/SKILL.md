---
name: single-cell-subpopulation-clustering
description: Use when scRNA-seq cells need to be grouped into subpopulations using
  either SNV-based features (eeSNVs from SSrGE) or raw gene-expression matrices, and
  when cluster quality must be benchmarked against known labels with AMI over bootstrap
  runs.
when_to_use_negative:
- The input is already a reduced/dimensionality-reduced feature table (e.g., pre-computed
  PCA components or a curated eeSNV matrix) — clustering can proceed directly without
  re-running the SSrGE feature derivation.
- The dataset is one of the four non-cancer scRNA-seq datasets (GSE79457, GSE81903,
  GSE71485, GSE80232) where the eeSNV-over-GE advantage was not the evaluated claim.
- No reference subpopulation labels are available — AMI cannot be computed, and the
  30-run bootstrap evaluation design is not applicable.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SSrGE
  role: Fits sparse LASSO linear models linking binary SNV matrix to gene expression,
    ranks eeSNVs, and reduces cells into eeSNV feature space via transform before
    clustering
  repo: https://github.com/lanagarmire/SSrGE.git
- name: scikit-learn 0.18
  role: Provides PCA, factor analysis, K-means, and agglomerative clustering (WARD
    linkage) used as four of the five clustering strategies
- name: SIMLR
  role: Fifth clustering strategy evaluated alongside the scikit-learn PCA/FA-based
    pipelines
- name: Numpy
  role: Python dependency for matrix operations in SSrGE and feature matrices
- name: Scipy
  role: Python dependency used by SSrGE for numerical routines
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-018-07170-5
    title: Using single nucleotide variations in single-cell RNA-seq to identify subpopulations
      and genotype-phenotype linkage
  source_content_hash: sha256:2df7923afb4729230aac6eea9b9f16e424a335d01bfe2e047b9fb4b4f0519b0a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/SSrGE.git
  tools:
  - Numpy
  - Python
  - SIMLR
  - SSrGE
  - Scipy
  - scikit-learn
  - scikit-learn 0.18
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# single-cell-subpopulation-clustering

## Summary

Clusters single-cell RNA-seq data into cancer cell subpopulations by comparing eeSNV-derived features against full gene-expression (GE) matrices across five clustering strategies (PCA/FA + K-means, PCA/FA + agglomerative WARD, and SIMLR), evaluated by adjusted mutual information (AMI) against known subpopulation labels.

## When to use

Use when scRNA-seq cells need to be grouped into subpopulations using either SNV-based features (eeSNVs from SSrGE) or raw gene-expression matrices, and when cluster quality must be benchmarked against known labels with AMI over bootstrap runs.

## When NOT to use

- The input is already a reduced/dimensionality-reduced feature table (e.g., pre-computed PCA components or a curated eeSNV matrix) — clustering can proceed directly without re-running the SSrGE feature derivation.
- The dataset is one of the four non-cancer scRNA-seq datasets (GSE79457, GSE81903, GSE71485, GSE80232) where the eeSNV-over-GE advantage was not the evaluated claim.
- No reference subpopulation labels are available — AMI cannot be computed, and the 30-run bootstrap evaluation design is not applicable.

## Inputs

- SNV matrix (n_cells x n_SNVs, binary) per dataset
- Gene expression matrix (n_cells x n_Genes, float) per dataset
- Known subpopulation labels for each cell (for AMI evaluation)
- GEO scRNA-seq datasets: GSE73121, GSE51372, GSE67980, GSE57872, GSE75688

## Outputs

- Reduced eeSNV feature matrices (n_cells x n_selected_eeSNVs) from ssrge.transform(X)
- Ranked eeSNV lists from ssrge.rank_eeSNVs()
- Cluster assignments per cell for each of the five clustering strategies on eeSNV and GE features
- AMI scores per bootstrap run per dataset, feature type, and clustering strategy
- Aggregated comparison table written to ami_comparison_results.csv

## How to apply

Download the five GEO scRNA-seq datasets (GSE73121, GSE51372, GSE67980, GSE57872, GSE75688) and build a binary SNV matrix (n_cells x n_SNVs, X_c,n = 1 if SNV s is present in cell c) and a gene-expression matrix (n_cells x n_Genes) for each. Fit SSrGE (Python 2.7, scikit-learn 0.18) on both matrices with ssrge.fit(X, Y), then derive eeSNV features via ssrge.rank_eeSNVs() and reduce the cells into eeSNV feature space with ssrge.transform(X). Cluster both the reduced eeSNV matrices and the full GE matrices with five strategies: PCA or factor-analysis dimensionality reduction followed by K-means, PCA/FA followed by agglomerative clustering with WARD linkage, and SIMLR. Repeat each clustering over 30 bootstrap resampling runs per dataset per feature type, computing adjusted mutual information (AMI, bounded in [0, 1]) against known subpopulation labels for every run, then aggregate mean and variability of AMI per dataset, feature type, and clustering strategy into ami_comparison_results.csv. The expected outcome is that eeSNV features achieve higher AMI than gene expression in all datasets tested, independent of the clustering method, reproducing the Fig. 2 pattern.

## Related tools

- **SSrGE** (Fits sparse LASSO linear models linking binary SNV matrix to gene expression, ranks eeSNVs, and reduces cells into eeSNV feature space via transform before clustering) — https://github.com/lanagarmire/SSrGE.git
- **scikit-learn 0.18** (Provides PCA, factor analysis, K-means, and agglomerative clustering (WARD linkage) used as four of the five clustering strategies)
- **SIMLR** (Fifth clustering strategy evaluated alongside the scikit-learn PCA/FA-based pipelines)
- **Numpy** (Python dependency for matrix operations in SSrGE and feature matrices)
- **Scipy** (Python dependency used by SSrGE for numerical routines)

## Examples

```
from garmire_SSrGE.ssrge import SSrGE; ssrge = SSrGE(); ssrge.fit(X, Y); X_r = ssrge.transform(X); ranked_feature = ssrge.rank_eeSNVs()
```

## Evaluation signals

- Mean AMI over 30 bootstrap runs for eeSNV features exceeds mean AMI for GE features in every one of the five datasets (Fig. 2 pattern reproduced)
- All AMI scores fall within the valid [0, 1] range
- The eeSNV advantage holds across all five clustering strategies (PCA/FA + K-means, PCA/FA + WARD, SIMLR), not just one
- X_r from ssrge.transform(X) has shape (n_cells, n_selected_eeSNVs) with n_selected_eeSNVs < n_SNVs, confirming successful feature reduction
- ssrge.test (python2 test/test_ssrge.py -v) passes, confirming a functional SSrGE installation under Python 2.7 and scikit-learn 0.18

## Limitations

- SSrGE may miss some driver mutations due to the incomplete DNA coverage of scRNA-seq reads.
- LASSO may arbitrarily select one of several highly correlated SNV features corresponding to gene expression, affecting feature stability.
- Reasons for the eeSNV accuracy advantage over GE remain speculative and are not experimentally verified.
- The GE feature space is subject to significant noise including batch effects and cell-cycle-dependent variation, which can confound clustering comparisons.

## Evidence

- [abstract] eeSNVs are consistently better features than GE across clustering methods: "In all the datasets tested, eeSNVs achieve better accuracies than gene expression for identifying subpopulations."
- [readme] SSrGE usage for fitting and transforming: "ssrge.fit(X, Y)

score_models, score_null_models = ssrge.score(X, Y)

X_r = ssrge.transform(X)"
- [readme] SNV matrix encoding procedure: "if a given snv (s) is present into a given cell (c), then X_c,n = 1"
- [discussion] LASSO as feature selection minimizing spurious SNVs: "Secondly, LASSO penalization works as a feature selection method and minimizes the spurious SNVs (false positive) from the filtered set of eeSNVs."
- [discussion] SSrGE may miss driver mutations: "Nevertheless, SSrGE may miss some driver mutations due to the incomplete DNA coverage of scRNA-seq reads."
