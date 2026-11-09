---
name: snv-gene-linkage-modeling-with-lasso
description: Use when you have a binary SNV matrix (n_cells x n_SNVs) and a gene expression
  matrix (n_cells x n_genes) from the same single cells and need to infer which eeSNVs
  are associated with expression of particular genes.
when_to_use_negative:
- You only have gene expression without SNV calls from the same single cells — SSrGE
  requires the coupled SNV matrix as features
- Input is already a ranked feature table or you need generic dimension reduction
  unrelated to SNV-gene linkage; LASSO may arbitrarily select one of highly correlated
  SNV features
- Raw reads without preprocessing — SNVs must first be identified from scRNA-seq raw
  reads via the SNV-calling pipeline before fitting
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3518
tools:
- name: SSrGE
  role: Python package that fits the sparse LASSO models linking eeSNVs to gene expression,
    performs cross-validation, parallel fitting, and eeSNV/gene ranking
  repo: https://github.com/lanagarmire/SSrGE.git
- name: scikit-learn
  role: Underlying library (version 0.18) providing the LASSO and ElasticNet linear
    models used by SSrGE
- name: Numpy
  role: Numerical array handling for the SNV and gene expression matrices
- name: Scipy
  role: Scientific computing dependency of the SSrGE package
- name: tabulate
  role: Tabular output formatting of SSrGE results such as ranked genes
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-018-07170-5
    title: Using single nucleotide variations in single-cell RNA-seq to identify subpopulations
      and genotype-phenotype linkage
  source_content_hash: sha256:28ecbf64343d66f6d415affcfe985bea369b3eb5df141292db1352284ea8f691
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
  - tabulate
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# snv-gene-linkage-modeling-with-lasso

## Summary

Fit sparse LASSO linear models (via the SSrGE Python package) that link filtered, effective, and expressed SNVs (eeSNVs) from single-cell RNA-seq data to gene expression profiles, enabling eeSNV/gene ranking and genotype-phenotype linkage.

## When to use

Use when you have a binary SNV matrix (n_cells x n_SNVs) and a gene expression matrix (n_cells x n_genes) from the same single cells and need to infer which eeSNVs are associated with expression of particular genes.

## When NOT to use

- You only have gene expression without SNV calls from the same single cells — SSrGE requires the coupled SNV matrix as features
- Input is already a ranked feature table or you need generic dimension reduction unrelated to SNV-gene linkage; LASSO may arbitrarily select one of highly correlated SNV features
- Raw reads without preprocessing — SNVs must first be identified from scRNA-seq raw reads via the SNV-calling pipeline before fitting

## Inputs

- Binary SNV matrix X (n_cells x n_SNVs), encoded as X_c,n = 1 if SNV n is present in cell c
- Gene expression matrix Y (n_cells x n_genes, float values)
- Optional CNV matrix C (n_cells x n_genes) describing CNV level per gene
- snv_id_list: tuples of (gene harboring the SNV, user-defined SNV id such as genomic position)
- gene_id_list ordered to correspond to the gene expression matrix columns
- Per-cell .vcf files (snv_filtered.vcf) and STAR gene expression files (matrix_counts.txt) if building matrices from raw outputs

## Outputs

- Sparse per-gene LASSO models retaining only non-null inferred coefficients
- Ranked eeSNV list from ssrge.rank_eeSNVs() based on accumulative coefficients
- Ranked gene list from ssrge.rank_genes()
- Reduced feature matrix X_r from ssrge.transform(X) restricted to selected eeSNVs
- Subgroup-specific ranked/significant genes and eeSNVs from rank_features_for_a_subgroup()

## How to apply

Instantiate SSrGE with an snv_id_list (each SNV id a tuple of the gene harboring the SNV and a user-defined SNV identifier such as genomic position) and a gene_id_list matching the gene expression matrix columns. Fit one sparse LASSO model per gene with ssrge.fit(X, Y), where X is the binary SNV matrix (X_c,n = 1 if SNV n is present in cell c) and Y is the gene expression matrix; LASSO penalization acts as a feature selection method that minimizes spurious (false positive) SNVs from the filtered eeSNV set. Optionally pass a CNV matrix C of shape (n_cells x n_genes) as a third argument, ssrge.fit(X, Y, C), to adjust the models for copy-number effects. Call ssrge.rank_eeSNVs() and ssrge.rank_genes() to obtain accumulative-coefficient rankings of eeSNVs and their harboring genes; the regularization parameter alpha can be replaced by nb_ranked_features to fix the number of retained eeSNVs, and the package supports parallelization and cross-validation via LinearCrossVal over the alpha regularization path.

## Related tools

- **SSrGE** (Python package that fits the sparse LASSO models linking eeSNVs to gene expression, performs cross-validation, parallel fitting, and eeSNV/gene ranking) — https://github.com/lanagarmire/SSrGE.git
- **scikit-learn** (Underlying library (version 0.18) providing the LASSO and ElasticNet linear models used by SSrGE)
- **Numpy** (Numerical array handling for the SNV and gene expression matrices)
- **Scipy** (Scientific computing dependency of the SSrGE package)
- **tabulate** (Tabular output formatting of SSrGE results such as ranked genes)

## Examples

```
from garmire_SSrGE.ssrge import SSrGE; ssrge = SSrGE(snv_id_list=snv_id_list, gene_id_list=gene_id_list, alpha=0.01); ssrge.fit(X, Y); print ssrge.rank_genes()
```

## Evaluation signals

- ssrge.score(X, Y) returns score_models vs. score_null_models — model scores should exceed null-model scores for informative eeSNV-gene links
- Gene rankings reproduce published Table 2 positions: HLA-A ranked 3rd (Kim) and 28th (Patel), B2M 7th (Ting) and 45th (Patel), KRAS 13th (Kim), AR 6th and KLK3 19th (Miyamoto), EGFR 88th of 4225 genes (Patel)
- Kendall-Tau correlation between gene rankings with and without the CNV matrix should be close to 1 (p = 0), confirming CNV-invariant ranking stability
- transform() output X_r should have exactly nb_ranked_features columns when that argument is set (e.g. shape (100, 2) for nb_ranked_features=2)
- LinearCrossVal regularization_path over alpha values (e.g. [0.01, 0.1, 0.2]) should complete without error, indicating well-formed matrices

## Limitations

- SSrGE may miss some driver mutations due to incomplete DNA coverage of scRNA-seq reads
- In case of correlated features, LASSO may select one of those highly correlated SNV features that correspond to gene expression, arbitrarily
- Requires Python 2 (>=2.7); scikit-learn version 0.18 is specified, limiting compatibility with modern environments

## Evidence

- [discussion] LASSO penalization used as a feature selection method to minimize spurious SNVs: "Secondly, LASSO penalization works as a feature selection method and minimizes the spurious SNVs (false positive) from the filtered set of eeSNVs."
- [discussion] Accumulative ranking of eeSNVs and harboring genes: "SSrGE uses an accumulative ranking approach to select eeSNVs linked to the expression of a particular gene."
- [readme] SNV id structure for gene ranking: "a SNV id should be a tuple containing the gene id harboring the given SNV and a user defined SNV id (genome position for example)."
- [readme] Optional CNV matrix input to fit: "The fit method can take an additional CNV matrix of shape (n_cells x n_genes), and describing the CNV level for each gene."
- [abstract] Cancer-relevant genes highly ranked: "Previously validated cancer-relevant genes are also highly ranked, confirming the significance of the method."
