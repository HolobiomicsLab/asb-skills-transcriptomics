---
name: dimensionality-reduction-with-kmeans-and-ward-agglomeration
description: Use when comparing how well alternative feature spaces (eeSNV vs gene
  expression) retrieve known cancer cell subpopulations in scRNA-seq data, or when
  benchmarking clustering pipelines (PCA/FA + K-means, PCA/FA + agglomerative WARD,
  SIMLR) with AMI.
when_to_use_negative:
- The goal is genotype-phenotype linkage or driver-gene ranking rather than subpopulation
  clustering — use SSrGE's rank_genes or rank_features_for_a_subgroup workflows instead.
- Input is raw scRNA-seq FASTQ/SRA data rather than already-built SNV and gene-expression
  matrices — first run the SSrGE download/SNV-calling pipeline (e.g., download_sra,
  extract_sra, STAR alignment).
- Only one feature space and one clustering method are of interest with no AMI benchmarking
  against known labels — the bootstrap AMI comparison design is unnecessary overhead.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0203
tools:
- name: SSrGE
  role: Fits LASSO sparse linear models linking SNVs to gene expression and transforms
    cells into reduced eeSNV feature space that is subsequently clustered; also ranks
    eeSNVs via ssrge.rank_eeSNVs().
  repo: https://github.com/lanagarmire/SSrGE.git
- name: scikit-learn
  role: Provides PCA, factor analysis, K-means, and agglomerative clustering with
    WARD linkage used as the five clustering strategies, plus adjusted mutual information
    scoring (version 0.18 in the original study).
- name: Python 2.7
  role: Runtime environment for SSrGE and the clustering/benchmarking scripts (python
    2.7.X must be used per the requirements).
- name: Numpy
  role: Numerical library requirement for SSrGE matrix operations (SNV and gene expression
    matrices).
- name: Scipy
  role: Scientific computing dependency for SSrGE and downstream clustering analyses.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-018-07170-5
    title: Using single nucleotide variations in single-cell RNA-seq to identify subpopulations
      and genotype-phenotype linkage
  source_content_hash: sha256:8d40e571f7d5da5d8e05d1c76939bbaa100629cfa043577cd204a607402fe834
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/SSrGE.git
  tools:
  - Numpy
  - Python
  - Python 2.7
  - SSrGE
  - Scipy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# Dimensionality-reduction with K-means and WARD agglomeration

## Summary

Cluster single cells by combining PCA or factor-analysis dimensionality reduction with K-means or agglomerative WARD-linkage clustering (plus SIMLR), applied to eeSNV and gene-expression features, and score subpopulation retrieval with adjusted mutual information (AMI) over 30 bootstrap runs.

## When to use

Use when comparing how well alternative feature spaces (eeSNV vs gene expression) retrieve known cancer cell subpopulations in scRNA-seq data, or when benchmarking clustering pipelines (PCA/FA + K-means, PCA/FA + agglomerative WARD, SIMLR) with AMI.

## When NOT to use

- The goal is genotype-phenotype linkage or driver-gene ranking rather than subpopulation clustering — use SSrGE's rank_genes or rank_features_for_a_subgroup workflows instead.
- Input is raw scRNA-seq FASTQ/SRA data rather than already-built SNV and gene-expression matrices — first run the SSrGE download/SNV-calling pipeline (e.g., download_sra, extract_sra, STAR alignment).
- Only one feature space and one clustering method are of interest with no AMI benchmarking against known labels — the bootstrap AMI comparison design is unnecessary overhead.

## Inputs

- SNV matrix (n_cells x n_SNVs, binary) per dataset
- Gene expression matrix (n_cells x n_Genes, float) per dataset
- SSrGE-transformed eeSNV feature matrix (from ssrge.transform(X))
- Known subpopulation labels per cell (for AMI computation)
- Five GEO scRNA-seq datasets: GSE73121, GSE51372, GSE67980, GSE57872, GSE75688

## Outputs

- Cluster assignments of cells for each combination of dataset, feature type, and clustering strategy
- Per-run adjusted mutual information (AMI) scores against known subpopulation labels over 30 bootstrap runs
- Aggregated mean and variability of AMI per dataset, feature type, and clustering strategy
- Full comparison table written to ami_comparison_results.csv

## How to apply

Build, per GEO dataset (e.g., GSE73121, GSE51372, GSE67980, GSE57872, GSE75688), a binary SNV matrix (n_cells x n_SNVs) and a gene-expression matrix (n_cells x n_Genes), then fit SSrGE (Python 2.7, scikit-learn 0.18) with ssrge.fit(X, Y) and derive ranked eeSNV features via ssrge.rank_eeSNVs(); transform cells into eeSNV feature space with ssrge.transform(X). Run five clustering strategies on both eeSNV and full GE matrices using scikit-learn: PCA or factor analysis followed by K-means, PCA or factor analysis followed by agglomerative clustering with WARD linkage, and SIMLR. Repeat each clustering over 30 bootstrap resampling runs per dataset and feature type, computing adjusted mutual information against the known subpopulation labels for every run. Aggregate mean and variability of AMI per dataset, feature type, and clustering strategy, and write the full comparison table to ami_comparison_results.csv; the aggregated AMI values should reproduce the Fig. 2 pattern in which eeSNV features achieve higher accuracy than gene expression in all datasets tested, with AMI bounded in [0, 1].

## Related tools

- **SSrGE** (Fits LASSO sparse linear models linking SNVs to gene expression and transforms cells into reduced eeSNV feature space that is subsequently clustered; also ranks eeSNVs via ssrge.rank_eeSNVs().) — https://github.com/lanagarmire/SSrGE.git
- **scikit-learn** (Provides PCA, factor analysis, K-means, and agglomerative clustering with WARD linkage used as the five clustering strategies, plus adjusted mutual information scoring (version 0.18 in the original study).)
- **Python 2.7** (Runtime environment for SSrGE and the clustering/benchmarking scripts (python 2.7.X must be used per the requirements).)
- **Numpy** (Numerical library requirement for SSrGE matrix operations (SNV and gene expression matrices).)
- **Scipy** (Scientific computing dependency for SSrGE and downstream clustering analyses.)

## Examples

```
from garmire_SSrGE.ssrge import SSrGE; ssrge = SSrGE(); ssrge.fit(X, Y); X_r = ssrge.transform(X); ranked_feature = ssrge.rank_eeSNVs()
```

## Evaluation signals

- Aggregated AMI values reproduce the Fig. 2 pattern: eeSNV features achieve higher accuracy than gene expression for identifying subpopulations in all datasets tested.
- All AMI scores fall within the valid range [0, 1].
- Each dataset/feature-type/clustering-strategy combination has 30 bootstrap AMI runs with aggregated mean and variability reported.
- X_r = ssrge.transform(X) yields a reduced feature matrix whose dimensions are consistent with the chosen number of eeSNVs (e.g., nb_ranked_features), verifiable via X_r.shape.
- The full comparison table is written to ami_comparison_results.csv with one entry per dataset, feature type, and clustering strategy.

## Limitations

- SSrGE may miss some driver mutations due to the incomplete DNA coverage of scRNA-seq reads.
- In case of correlated features, LASSO may arbitrarily select one of those highly correlated SNV features that correspond to gene expression.
- Gene expression features in scRNA-seq data are subject to significant noise, including batch effects and cell-cycle-dependent variation, which affects the GE clustering baseline.
- Explanations for why eeSNVs outperform GE features are speculative (not experimentally verified) per the article's discussion.

## Evidence

- [abstract] eeSNVs achieve better accuracies than gene expression for identifying subpopulations in all datasets tested: "In all the datasets tested, eeSNVs achieve better accuracies than gene expression for identifying subpopulations."
- [discussion] LASSO penalization minimizes spurious SNVs: "Secondly, LASSO penalization works as a feature selection method and minimizes the spurious SNVs (false positive) from the filtered set of eeSNVs."
- [readme] SSrGE takes SNV and gene matrices as input: "SSrGE takes as input two matrices (A SNV matrix (n_cells x n_SNVs) and a Gene matrix (n_cells x n_Genes)"
- [methods] scikit-learn 0.18 requirement: "[Scikit-learn](http://scikit-learn.org/) (version = 0.18)"
