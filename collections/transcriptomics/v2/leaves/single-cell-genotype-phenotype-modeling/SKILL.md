---
name: single-cell-genotype-phenotype-modeling
description: Use when you have a binary SNV matrix (n_cells x n_SNVs) from scRNA-seq
  or coupled DNA-/RNA-seq single cells and a gene expression matrix (n_cells x n_Genes),
  and want expression-linked SNV features (eeSNVs) for subclustering or genotype markers.
when_to_use_negative:
- When you already have a reduced feature table or need no SNV-expression linkage
  — SSrGE is a dimension reduction/feature selection step over raw SNV matrices, not
  a general clustering tool
- When SNVs and expression come from different (non-coupled) cells or when scRNA-seq
  DNA coverage is too incomplete to call SNVs reliably — SSrGE may miss driver mutations
  due to incomplete DNA coverage of scRNA-seq reads
- When SNV features are highly correlated and you need all of them retained — LASSO
  may arbitrarily select one of highly correlated SNV features corresponding to GE
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: SSrGE
  role: Python package that fits per-gene LASSO sparse linear models on the SNV and
    gene expression matrices, transforms cells into the eeSNV reduced representation,
    and ranks eeSNVs/genes
  repo: https://github.com/lanagarmire/SSrGE.git
- name: scikit-learn 0.18
  role: Underlying library providing the LASSO (and optional ElasticNet with l1_ratio=0.5)
    estimators used by SSrGE
- name: Numpy
  role: Numerical backend for the SNV and gene expression matrices
- name: Scipy
  role: Required Python dependency of the SSrGE package
- name: tabulate
  role: Required Python dependency used to format SSrGE output tables
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-018-07170-5
    title: Using single nucleotide variations in single-cell RNA-seq to identify subpopulations
      and genotype-phenotype linkage
  source_content_hash: sha256:5e0af5bcd60ca2e053f153e18fa4b2cbdb106c96281d09990d779f6d73e1cb69
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
  - tabulate
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# single-cell-genotype-phenotype-modeling

## Summary

Fit sparse (LASSO-regularized) linear models regressing single-cell gene expression on binary SNV matrices to infer eeSNVs — effective and expressed SNVs linked to expression — for subpopulation identification and genotype-phenotype linkage in scRNA-seq data.

## When to use

Use when you have a binary SNV matrix (n_cells x n_SNVs) from scRNA-seq or coupled DNA-/RNA-seq single cells and a gene expression matrix (n_cells x n_Genes), and want expression-linked SNV features (eeSNVs) for subclustering or genotype markers.

## When NOT to use

- When you already have a reduced feature table or need no SNV-expression linkage — SSrGE is a dimension reduction/feature selection step over raw SNV matrices, not a general clustering tool
- When SNVs and expression come from different (non-coupled) cells or when scRNA-seq DNA coverage is too incomplete to call SNVs reliably — SSrGE may miss driver mutations due to incomplete DNA coverage of scRNA-seq reads
- When SNV features are highly correlated and you need all of them retained — LASSO may arbitrarily select one of highly correlated SNV features corresponding to GE

## Inputs

- Binary SNV matrix X (n_cells x n_SNVs), encoded as X_c,n = 1 if SNV s is present in cell c (continuous encodings, e.g. 1 for 1/1 and 0.5 for 0/1 genotype, are also supported)
- Gene expression matrix Y (n_cells x n_Genes, float values per cell)
- Optional CNV matrix C (n_cells x n_genes) describing per-gene CNV levels
- Optional snv_id_list (tuples of gene id + SNV id, e.g. genome position) and gene_id_list aligned to matrix columns/rows
- Per-cell .vcf files (snv_filtered.vcf) and STAR matrix_counts.txt expression files if building matrices from scratch

## Outputs

- Reduced eeSNV feature matrix X_r (n_cells x n_eeSNVs) from ssrge.transform(X)
- Ranked eeSNV list from ssrge.rank_eeSNVs() and ranked genes from ssrge.rank_genes()
- Model scores: score_models and score_null_models from ssrge.score(X, Y)
- Subgroup-specific eeSNVs and impacted genes from ssrge.rank_features_for_a_subgroup(cell_indices), with ranked_genes, ranked_eeSNVs, significant_genes, significant_eeSNVs
- Regularization-path cross-validation results from LinearCrossVal.regularization_path

## How to apply

Instantiate the SSrGE class with the LASSO model and a regularization parameter alpha (e.g. 0.01), running under Python 2.7 with Numpy, Scipy, and scikit-learn 0.18. Call ssrge.fit(X, Y) on the SNV matrix (n_cells x n_SNVs, binary encoding: X_c,n = 1 if SNV s is present in cell c) and the gene expression matrix (n_cells x n_Genes); optionally pass a CNV matrix C (n_cells x n_genes) as a third argument. The procedure fits one sparse linear model per gene and keeps the non-null inferred coefficients, so an SNV's score is the sum of its inferred coefficients and a gene's score is the sum of its SNV scores — LASSO penalization acts as feature selection that minimizes spurious (false positive) SNVs. Score the fitted models against the data with ssrge.score(X, Y) to obtain score_models and score_null_models, then transform X into the reduced eeSNV representation with ssrge.transform(X) and rank eeSNVs with ssrge.rank_eeSNVs() (or fix the count with nb_ranked_features instead of alpha). Optionally validate the regularization choice with LinearCrossVal.regularization_path('alpha', [0.01, 0.1, 0.2]) from garmire_SSrGE.linear_cross_validation, and write ranked eeSNV/gene outputs (e.g. to ssrge_ranked_eesnvs.tsv).

## Related tools

- **SSrGE** (Python package that fits per-gene LASSO sparse linear models on the SNV and gene expression matrices, transforms cells into the eeSNV reduced representation, and ranks eeSNVs/genes) — https://github.com/lanagarmire/SSrGE.git
- **scikit-learn 0.18** (Underlying library providing the LASSO (and optional ElasticNet with l1_ratio=0.5) estimators used by SSrGE)
- **Numpy** (Numerical backend for the SNV and gene expression matrices)
- **Scipy** (Required Python dependency of the SSrGE package)
- **tabulate** (Required Python dependency used to format SSrGE output tables)

## Examples

```
from garmire_SSrGE.ssrge import SSrGE; ssrge = SSrGE(alpha=0.01); ssrge.fit(X, Y); X_r = ssrge.transform(X); ranked_feature = ssrge.rank_eeSNVs()
```

## Evaluation signals

- ssrge.score(X, Y) returns score_models and score_null_models; fitted models should outperform null models as a sanity check
- X_r.shape from ssrge.transform(X) should be (n_cells, n_eeSNVs) with n_eeSNVs <= n_SNVs, and (100, 2) when nb_ranked_features=2 is set on the example matrices
- rank_eeSNVs() should return a ranked feature list; with snv_id_list/gene_id_list supplied, rank_genes() should attribute eeSNV scores to their harboring genes (e.g. KRAS, HLA-A, SPARC)
- LinearCrossVal.regularization_path('alpha', [0.01, 0.1, 0.2]) should produce comparable cross-validated scores across candidate alphas to justify the choice of 0.01
- python2 test/test_ssrge.py -v should pass, confirming the installation is functional

## Limitations

- SSrGE may miss some driver mutations due to the incomplete DNA coverage of scRNA-seq reads
- In case of correlated features, LASSO may select one of those highly correlated SNV features that correspond to GE, dropping the others
- The original pipeline requires Python 2.7 (>=2.7) with scikit-learn exactly at version 0.18; newer environments are not supported out of the box
- Gene scores are sums of SNV scores, so interpretations of ranked genes are indirect proxies rather than direct expression effects

## Evidence

- [discussion] LASSO penalization as per-gene feature selection: "Secondly, LASSO penalization works as a feature selection method and minimizes the spurious SNVs (false positive) from the filtered set of eeSNVs."
- [methods] Package usage as python package with fit(X, Y): "SSrGE should be used as a python package, below are usage example.
SSrGE takes as input two matrices (A SNV matrix (n_cells x n_SNVs) and a Gene matrix (n_cells x n_Genes)"
- [readme] Sparse LASSO model per gene keeping non-null coefficients: "The procedure infers a sparse linear model (LASSO by default) for each gene (raw in the second matrix) and keeps the non-null inferred coefs."
- [readme] Cross-validation of the alpha regularization parameter: "path = cross_val.regularization_path('alpha',  [0.01, 0.1, 0.2])"
- [abstract] eeSNVs outperform gene expression for subpopulations: "In all the datasets tested, eeSNVs achieve better accuracies than gene expression for identifying subpopulations."
- [discussion] Accumulative ranking of eeSNVs linked to gene expression: "SSrGE uses an accumulative ranking approach to select eeSNVs linked to the expression of a particular gene."
