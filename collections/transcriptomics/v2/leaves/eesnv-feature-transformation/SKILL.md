---
name: eesnv-feature-transformation
description: Use when you have paired per-cell SNV (n_cells x n_SNVs binary) and gene
  expression (n_cells x n_Genes) matrices from scRNA-seq and need filtered, effective
  SNV features instead of noisy gene expression for subpopulation analysis.
when_to_use_negative:
- You only have gene expression data and no SNV calls per cell — SSrGE requires the
  paired SNV matrix as the feature matrix X
- You need a general dimension-reduction method for any matrix unrelated to SNV–gene
  expression linkage — SSrGE is designed to link SNVs to GE via sparse linear models
- You require detection of all driver mutations — SSrGE may miss driver mutations
  due to the incomplete DNA coverage of scRNA-seq reads
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_3518
tools:
- name: SSrGE
  role: Python package that fits sparse LASSO models per gene on the SNV feature matrix
    against the gene expression response, keeps non-null coefficients, and transforms
    cells into eeSNV feature space via ssrge.transform(X) with rank_eeSNVs() and rank_genes()
    rankings
  repo: https://github.com/lanagarmire/SSrGE.git
- name: scikit-learn 0.18
  role: Underlying library providing the LASSO (and optional ElasticNet with l1_ratio)
    sparse linear models used for eeSNV selection
- name: Python 2.7
  role: Required interpreter version for running the SSrGE package and its requirements
- name: Numpy
  role: Array handling for the SNV and gene expression matrices passed to ssrge.fit
    and transformed to X_r
- name: Scipy
  role: Supporting scientific library required by the SSrGE package
- name: tabulate
  role: Python dependency used to tabulate SSrGE outputs such as ranked features
provenance:
  source_task_ids:
  - task_004
  - task_007
  source_papers:
  - doi: 10.1038/s41467-018-07170-5
    title: Using single nucleotide variations in single-cell RNA-seq to identify subpopulations
      and genotype-phenotype linkage
  source_content_hash: sha256:cb9fe8d9e344a4624eafbc7ae1e27884293bd1f18c4963403fe17884c5110499
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
  - tabulate
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# eesnv-feature-transformation

## Summary

Transforms a per-cell binary SNV matrix into a reduced eeSNV (effective and expressed SNV) feature space by fitting sparse LASSO models that link each SNV to gene expression with the SSrGE package, yielding denoised genetic features for downstream clustering, MST pseudo-time, and ranking.

## When to use

Use when you have paired per-cell SNV (n_cells x n_SNVs binary) and gene expression (n_cells x n_Genes) matrices from scRNA-seq and need filtered, effective SNV features instead of noisy gene expression for subpopulation analysis.

## When NOT to use

- You only have gene expression data and no SNV calls per cell — SSrGE requires the paired SNV matrix as the feature matrix X
- You need a general dimension-reduction method for any matrix unrelated to SNV–gene expression linkage — SSrGE is designed to link SNVs to GE via sparse linear models
- You require detection of all driver mutations — SSrGE may miss driver mutations due to the incomplete DNA coverage of scRNA-seq reads

## Inputs

- SNV matrix X (n_cells x n_SNVs, binary or genotype-dosage encoding)
- Gene expression matrix Y (n_cells x n_Genes, float counts/expression)
- Optional CNV matrix (n_cells x n_genes) for the fit method
- Optional snv_id_list (tuples of harboring gene id and SNV id) and gene_id_list for gene ranking

## Outputs

- Reduced eeSNV feature matrix X_r returned by ssrge.transform(X)
- Ranked eeSNV list from ssrge.rank_eeSNVs()
- Ranked and significant genes from ssrge.rank_genes()
- Subgroup-specific eeSNVs/genes from ssrge.rank_features_for_a_subgroup(cell_indices)

## How to apply

Instantiate SSrGE and call ssrge.fit(X, Y) on the SNV matrix X (binary encoding: X_c,n = 1 if SNV n is present in cell c; genotype dosages like 1/1 = 1 and 0/1 = 0.5 are also allowed) and the gene expression matrix Y; an optional CNV matrix (n_cells x n_genes) can be passed to fit. The default model is LASSO, which acts as feature selection minimizing spurious SNVs, and the number of retained eeSNVs is controlled by the alpha regularization parameter or, alternatively, by nb_ranked_features to fix a top-k count. Call ssrge.transform(X) to obtain the reduced matrix X_r in eeSNV space, then ssrge.rank_eeSNVs() to get the ranked eeSNV features (rank_genes() with supplied snv_id_list and gene_id_list to rank harboring genes). Use Python 2.7 with scikit-learn 0.18, Numpy, Scipy, and tabulate per the package requirements.

## Related tools

- **SSrGE** (Python package that fits sparse LASSO models per gene on the SNV feature matrix against the gene expression response, keeps non-null coefficients, and transforms cells into eeSNV feature space via ssrge.transform(X) with rank_eeSNVs() and rank_genes() rankings) — https://github.com/lanagarmire/SSrGE.git
- **scikit-learn 0.18** (Underlying library providing the LASSO (and optional ElasticNet with l1_ratio) sparse linear models used for eeSNV selection)
- **Python 2.7** (Required interpreter version for running the SSrGE package and its requirements)
- **Numpy** (Array handling for the SNV and gene expression matrices passed to ssrge.fit and transformed to X_r)
- **Scipy** (Supporting scientific library required by the SSrGE package)
- **tabulate** (Python dependency used to tabulate SSrGE outputs such as ranked features)

## Examples

```
from garmire_SSrGE.ssrge import SSrGE; ssrge = SSrGE(); ssrge.fit(X, Y); X_r = ssrge.transform(X); ranked_feature = ssrge.rank_eeSNVs()
```

## Evaluation signals

- X_r returned by ssrge.transform(X) must have shape (n_cells, n_eeSNVs), with n_eeSNVs controlled by alpha or nb_ranked_features (e.g., SSrGE(nb_ranked_features=2) gives X_r.shape == (100, 2) on the example matrix)
- ssrge.rank_eeSNVs() returns a non-empty ranked feature list whose top eeSNVs and harboring genes include previously validated cancer-relevant genes (e.g., B2M in Ting/Patel datasets)
- Downstream clustering on eeSNV features must outperform clustering on full gene expression, with AMI scores bounded in [0, 1] across 30 bootstrap runs per dataset
- ssrge.score(X, Y) should yield score_models that outperform score_null_models, confirming the sparse models capture SNV–expression linkage
- rank_features_for_a_subgroup reports eeSNVs significantly more present among the target subgroup's cells (non-empty significant_eeSNVs)

## Limitations

- SSrGE may miss some driver mutations due to the incomplete DNA coverage of scRNA-seq reads
- In case of correlated features, LASSO may arbitrarily select one of highly correlated SNV features corresponding to gene expression
- Requires the legacy stack: Linux, Python 2.7.x, and scikit-learn 0.18

## Evidence

- [abstract] We develop a linear modeling framework, SSrGE, to link eeSNVs associated with gene expression.: "We develop a linear modeling framework, SSrGE, to link eeSNVs associated with gene expression."
- [discussion] LASSO penalization minimizes spurious SNVs: "Secondly, LASSO penalization works as a feature selection method and minimizes the spurious SNVs (false positive) from the filtered set of eeSNVs."
- [readme] README encoding rule for X: "if a given snv (s) is present into a given cell (c), then X_c,n = 1"
- [readme] README usage example for fit/transform/rank: "ssrge.fit(X, Y)

X_r = ssrge.transform(X)

ranked_feature = ssrge.rank_eeSNVs()"
- [methods] nb_ranked_features as alternative to alpha: "Instead of relying on the regularization parameter (alpha), to select the number of eeSNVs, the `nb_ranked_features` argument can be specified"
