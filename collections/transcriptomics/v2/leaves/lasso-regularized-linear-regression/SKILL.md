---
name: lasso-regularized-linear-regression
description: Use when you have coupled single-cell SNV (or VCF-derived) and gene expression
  matrices and need sparse SNV-to-expression models, feature selection, and eeSNV/gene
  rankings for subpopulation or genotype-phenotype analysis.
when_to_use_negative:
- Inputs are already reduced features or a ranked feature table — SSrGE's transform/ranking
  operates on raw SNV x expression matrices
- The SNVs and gene expression come from different (non-coupled) cells — the framework
  relies on paired DNA/RNA measurements from the same single cells
- Highly correlated SNV features must all be retained — plain LASSO may arbitrarily
  select only one of them; use the ElasticNet variant instead
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: SSrGE
  role: Python package that instantiates and fits the LASSO sparse linear models,
    transforms the SNV matrix into eeSNV features, and ranks eeSNVs/genes
  repo: https://github.com/lanagarmire/SSrGE.git
- name: scikit-learn 0.18
  role: Provides the LASSO/ElasticNet linear models underlying SSrGE's fitting procedure
- name: Python 2.7
  role: Runtime environment required by the SSrGE package (installed via pip2 install
    -r requirements.txt)
- name: Numpy
  role: Numerical library dependency for the SSrGE matrix operations
- name: Scipy
  role: Scientific computing dependency for the SSrGE package
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-018-07170-5
    title: Using single nucleotide variations in single-cell RNA-seq to identify subpopulations
      and genotype-phenotype linkage
  source_content_hash: sha256:d62ef2b67f4fea7695b3822e14243579f91de0b0eb9c37fc89fddbe2f0d364b2
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
  - scikit-learn 0.18
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# lasso-regularized-linear-regression

## Summary

Fit LASSO (or ElasticNet) sparse linear models regressing gene expression on binary/encoded SNV matrices to infer eeSNVs, as implemented in the SSrGE Python package from the single-cell SNV-to-GE genotype-phenotype linkage framework.

## When to use

Use when you have coupled single-cell SNV (or VCF-derived) and gene expression matrices and need sparse SNV-to-expression models, feature selection, and eeSNV/gene rankings for subpopulation or genotype-phenotype analysis.

## When NOT to use

- Inputs are already reduced features or a ranked feature table — SSrGE's transform/ranking operates on raw SNV x expression matrices
- The SNVs and gene expression come from different (non-coupled) cells — the framework relies on paired DNA/RNA measurements from the same single cells
- Highly correlated SNV features must all be retained — plain LASSO may arbitrarily select only one of them; use the ElasticNet variant instead

## Inputs

- Binary or encoded SNV matrix X (n_cells x n_SNVs; X_c,n = 1 if SNV s is present in cell c, or 0.5/1 for 0/1 vs 1/1 genotypes)
- Gene expression matrix Y (n_cells x n_Genes, float values)
- Optional CNV matrix C (n_cells x n_genes) describing CNV level per gene
- Optional list of per-cell .vcf files (snv_filtered.vcf) and STAR matrix_counts.txt expression files to build the matrices

## Outputs

- Sparse per-gene LASSO coefficient sets (non-null inferred coefficients kept for each gene)
- score_models and score_null_models from ssrge.score(X, Y)
- Reduced eeSNV representation matrix X_r from ssrge.transform(X)
- Ranked eeSNVs (ssrge.rank_eeSNVs()) and ranked genes (ssrge.rank_genes()), e.g. written to ssrge_ranked_eesnvs.tsv
- Subgroup-specific ranked/significant eeSNVs and genes via ssrge.rank_features_for_a_subgroup()

## How to apply

Instantiate the SSrGE class (Python 2.7, scikit-learn 0.18, Numpy, Scipy, tabulate) with model='LASSO' and a regularization parameter alpha (e.g. 0.01), then call fit(X, Y) on the SNV matrix (n_cells x n_SNVs, binary encoding: 1 if the SNV is present in the cell) and gene expression matrix (n_cells x n_Genes); an optional CNV matrix C (n_cells x n_genes) can be passed as a third argument. LASSO penalization acts as feature selection, minimizing spurious (false positive) SNVs, and a gene's score is the sum of its SNV coefficient scores. Score models with ssrge.score(X, Y) to obtain score_models versus score_null_models, reduce X with ssrge.transform(X), and rank features with ssrge.rank_eeSNVs() and ssrge.rank_genes() (supplying snv_id_list tuples of (gene id, position) and gene_id_list to enable gene ranking). Alternatively fix the number of eeSNVs via nb_ranked_features instead of alpha, or validate the alpha choice with LinearCrossVal.regularization_path('alpha', [0.01, 0.1, 0.2]); ElasticNet (alpha, l1_ratio=0.5) is available when correlated SNV features make LASSO's arbitrary pick problematic.

## Related tools

- **SSrGE** (Python package that instantiates and fits the LASSO sparse linear models, transforms the SNV matrix into eeSNV features, and ranks eeSNVs/genes) — https://github.com/lanagarmire/SSrGE.git
- **scikit-learn 0.18** (Provides the LASSO/ElasticNet linear models underlying SSrGE's fitting procedure)
- **Python 2.7** (Runtime environment required by the SSrGE package (installed via pip2 install -r requirements.txt))
- **Numpy** (Numerical library dependency for the SSrGE matrix operations)
- **Scipy** (Scientific computing dependency for the SSrGE package)

## Examples

```
from garmire_SSrGE.ssrge import SSrGE; ssrge = SSrGE(model='LASSO', alpha=0.01); ssrge.fit(X, Y); score_models, score_null_models = ssrge.score(X, Y); X_r = ssrge.transform(X); ranked_feature = ssrge.rank_eeSNVs()
```

## Evaluation signals

- ssrge.score(X, Y) returns score_models that should exceed the corresponding score_null_models for informative fits
- X_r from ssrge.transform(X) has shape (n_cells, n_selected_eeSNVs); with nb_ranked_features=k the reduced matrix should have exactly k columns
- rank_eeSNVs() returns a ranked feature list whose length equals the number of non-null LASSO coefficients retained across genes
- Cross-validation over alpha values (e.g. [0.01, 0.1, 0.2]) via LinearCrossVal.regularization_path should show a coherent regularization path before committing to an alpha
- The test suite python2 test/test_ssrge.py -v runs without errors, confirming the installation is functional

## Limitations

- SSrGE may miss some driver mutations due to the incomplete DNA coverage of scRNA-seq reads
- In case of correlated features, LASSO may select one of those highly correlated SNV features that correspond to GE
- Python 2.7.X must be used, and scikit-learn must be version 0.18; the package requires a Linux working environment

## Evidence

- [abstract] SSrGE fits LASSO-regularized linear models: "We develop a linear modeling framework, SSrGE, to link eeSNVs associated with gene expression."
- [discussion] LASSO penalization as feature selection: "Secondly, LASSO penalization works as a feature selection method and minimizes the spurious SNVs (false positive) from the filtered set of eeSNVs."
- [discussion] LASSO correlated feature limitation: "but in case of correlated features, LASSO may select one of those highly correlated SNV features that correspond to GE"
- [readme] Package usage and input encoding: "SSrGE takes as input two matrices (A SNV matrix (n_cells x n_SNVs) and a Gene matrix (n_cells x n_Genes)"
- [readme] Fit and transform usage: "ssrge.fit(X, Y)

score_models, score_null_models = ssrge.score(X, Y)"
- [readme] nb_ranked_features alternative to alpha: "Instead of relying on the regularization parameter (alpha), to select the number of eeSNVs, the `nb_ranked_features` argument can be specified"
- [methods] Cross-validation of alpha: "path = cross_val.regularization_path('alpha',  [0.01, 0.1, 0.2])"
