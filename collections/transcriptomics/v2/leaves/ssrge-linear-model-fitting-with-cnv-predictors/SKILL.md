---
name: ssrge-linear-model-fitting-with-cnv-predictors
description: Use when you have per-cell SNV matrices and gene expression matrices
  from scRNA-seq (or coupled RRBS/RNA-seq) data and need to identify expressed SNVs
  (eeSNVs) associated with gene expression, with or without copy-number predictors.
when_to_use_negative:
- Input is already a reduced/ranked feature matrix — SSrGE is a feature selection
  step applied to raw SNV x cell and gene x cell matrices, not a downstream analysis
  tool
- SNVs are highly correlated and you need all of them retained — LASSO may arbitrarily
  select only one of highly correlated SNV features corresponding to a gene's expression
- The driver mutation has incomplete DNA coverage in the scRNA-seq reads — SSrGE may
  miss driver mutations entirely in that case
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0199
tools:
- name: SSrGE
  role: 'Core package: fits sparse (LASSO/ElasticNet) linear models per gene linking
    eeSNV features to expression, with optional CNV predictor and eeSNV/gene ranking'
  repo: https://github.com/lanagarmire/SSrGE.git
- name: scikit-learn
  role: Underlying library providing the LASSO and ElasticNet models (version 0.18)
  repo: http://scikit-learn.org/
- name: Python 2.7
  role: Required runtime for the SSrGE package (python 2.7.X must be used)
  repo: https://github.com/lanagarmire/SSrGE.git
- name: Numpy
  role: Numerical dependency for matrix handling in SSrGE
- name: Scipy
  role: Numerical dependency for SSrGE
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-018-07170-5
    title: Using single nucleotide variations in single-cell RNA-seq to identify subpopulations
      and genotype-phenotype linkage
  source_content_hash: sha256:c2c80ce0cd44f575e6fc3a46e45bbb3da92c5aa5e9620d2a6916d95f26b80b94
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
  - tabulate
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# ssrge-linear-model-fitting-with-cnv-predictors

## Summary

Fit SSrGE sparse linear models (LASSO by default, ElasticNet optional) linking binary eeSNV matrices (n_cells x n_SNVs) to gene expression matrices (n_cells x n_Genes), optionally including a CNV matrix (n_cells x n_genes) as an additional predictor to recover SNV-gene expression associations in single-cell data.

## When to use

Use when you have per-cell SNV matrices and gene expression matrices from scRNA-seq (or coupled RRBS/RNA-seq) data and need to identify expressed SNVs (eeSNVs) associated with gene expression, with or without copy-number predictors.

## When NOT to use

- Input is already a reduced/ranked feature matrix — SSrGE is a feature selection step applied to raw SNV x cell and gene x cell matrices, not a downstream analysis tool
- SNVs are highly correlated and you need all of them retained — LASSO may arbitrarily select only one of highly correlated SNV features corresponding to a gene's expression
- The driver mutation has incomplete DNA coverage in the scRNA-seq reads — SSrGE may miss driver mutations entirely in that case

## Inputs

- Binary SNV matrix X (n_cells x n_SNVs), encoded as 1 if SNV present (0.5 for 0/1 genotype, 1 for 1/1 genotype allowed)
- Gene expression matrix Y (n_cells x n_Genes), e.g. from STAR matrix_counts.txt files
- Optional CNV matrix C (n_cells x n_genes) describing CNV level per gene
- Optional snv_id_list (tuples of gene id + SNV id, e.g. genome position) and gene_id_list

## Outputs

- Fitted sparse linear model per gene with non-null SNV coefficients
- Reduced feature matrix X_r from ssrge.transform(X)
- Global eeSNV ranking from ssrge.rank_eeSNVs()
- Gene ranking from ssrge.rank_genes() when SNV/gene ids provided
- Model and null-model scores from ssrge.score(X, Y)

## How to apply

Instantiate SSrGE in Python 2.7 and fit with ssrge.fit(X, Y), where X is the SNV matrix encoded as X_c,n = 1 if SNV s is present in cell c (optionally 0.5 for heterozygous 0/1 and 1 for homozygous 1/1 genotypes) and Y is the gene expression matrix; pass an optional CNV matrix C (n_cells x n_genes) as a third argument via ssrge.fit(X, Y, C). Control sparsity with the regularization parameter (default LASSO, alpha=0.01) or switch to ElasticNet with SSrGE(model='ElasticNet', alpha=0.01, l1_ratio=0.5); alternatively, fix the number of selected eeSNVs with nb_ranked_features instead of relying on alpha. The procedure infers a sparse linear model per gene and keeps non-null coefficients, so LASSO penalization acts as feature selection that minimizes spurious (false positive) SNVs. After fitting, call ssrge.rank_eeSNVs() for a global eeSNV ranking and ssrge.score(X, Y) to compare model versus null-model fits; supply snv_id_list and gene_id_list to obtain gene-level rankings with ssrge.rank_genes().

## Related tools

- **SSrGE** (Core package: fits sparse (LASSO/ElasticNet) linear models per gene linking eeSNV features to expression, with optional CNV predictor and eeSNV/gene ranking) — https://github.com/lanagarmire/SSrGE.git
- **scikit-learn** (Underlying library providing the LASSO and ElasticNet models (version 0.18)) — http://scikit-learn.org/
- **Python 2.7** (Required runtime for the SSrGE package (python 2.7.X must be used)) — https://github.com/lanagarmire/SSrGE.git
- **Numpy** (Numerical dependency for matrix handling in SSrGE)
- **Scipy** (Numerical dependency for SSrGE)

## Examples

```
from garmire_SSrGE.ssrge import SSrGE; ssrge = SSrGE(model='ElasticNet', alpha=0.01, l1_ratio=0.5); ssrge.fit(X, Y, C); ranked_feature = ssrge.rank_eeSNVs()
```

## Evaluation signals

- ssrge.score(X, Y) returns score_models and score_null_models; fitted models should outperform null models
- With nb_ranked_features=k, ssrge.transform(X) yields a matrix with exactly k columns (README shows shape (100, 2) for nb_ranked_features=2)
- rank_eeSNVs() returns a non-empty ranked_feature list; the simplified top-ranked-features-only model should recover the ~92% eeSNV overlap reported for the Hou HCC dataset
- Re-fitting with model='ElasticNet', alpha=0.01, l1_ratio=0.5 should complete without error and produce comparable rankings
- Python 2.7 test suite passes: python2 test/test_ssrge.py -v

## Limitations

- LASSO may arbitrarily select one of highly correlated SNV features corresponding to gene expression
- SSrGE may miss some driver mutations due to the incomplete DNA coverage of scRNA-seq reads
- Python 2.7.X must be used (Python 2 is a requirement)
- Scikit-learn version 0.18 is specified

## Evidence

- [discussion] LASSO penalization works as a feature selection method and minimizes the spurious SNVs (false positive) from the filtered set of eeSNVs.: "Secondly, LASSO penalization works as a feature selection method and minimizes the spurious SNVs (false positive) from the filtered set of eeSNVs."
- [readme] The fit method can take an additional CNV matrix of shape (n_cells x n_genes), and describing the CNV level for each gene.: "The fit method can take an additional CNV matrix of shape (n_cells x n_genes), and describing the CNV level for each gene."
- [methods] Instead of relying on the regularization parameter (alpha), to select the number of eeSNVs, the `nb_ranked_features` argument can be specified: "Instead of relying on the regularization parameter (alpha), to select the number of eeSNVs, the `nb_ranked_features` argument can be specified"
- [abstract] We develop a linear modeling framework, SSrGE, to link eeSNVs associated with gene expression.: "We develop a linear modeling framework, SSrGE, to link eeSNVs associated with gene expression."
- [readme] if a given snv (s) is present into a given cell (c), then X_c,n = 1: "if a given snv (s) is present into a given cell (c), then X_c,n = 1"
