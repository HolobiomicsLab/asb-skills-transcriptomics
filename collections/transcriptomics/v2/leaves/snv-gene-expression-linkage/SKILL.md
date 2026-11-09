---
name: snv-gene-expression-linkage
description: Use when you have per-cell SNV calls (e.g. .vcf-derived binary matrix)
  coupled with single-cell gene expression counts and need SNV-derived features that
  link genotype to expression phenotypes.
when_to_use_negative:
- Input is already a reduced feature table or the SNVs and expression come from different
  cells — SSrGE requires coupled DNA/RNA data from the same single cells
- The goal is de novo SNV calling from raw reads — use the SSrGE SNV_calling/download
  modules or QuASAR first; this skill starts from already-called SNVs
- All candidate SNV features are highly correlated — LASSO may arbitrarily select
  only one of highly correlated SNV features corresponding to gene expression
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: SSrGE
  role: Python package that fits per-gene LASSO sparse linear models, transforms the
    SNV matrix into eeSNV features, and performs eeSNV/gene rankings
  repo: https://github.com/lanagarmire/SSrGE.git
- name: scikit-learn 0.18
  role: Underlying library providing the LASSO (and optional ElasticNet) linear models
    used by SSrGE
- name: QuASAR
  role: Upstream R package for genotyping from sequencing reads and inferring allelic
    imbalance at heterozygous sites, feeding SNV calls into the linkage stage
  repo: piquelab/QuASAR
- name: Python 2.7
  role: Runtime environment (with Numpy, Scipy, tabulate) in which the garmire_SSrGE
    package is installed and executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-018-07170-5
    title: Using single nucleotide variations in single-cell RNA-seq to identify subpopulations
      and genotype-phenotype linkage
  source_content_hash: sha256:8c6471bdc195be432b64372d121997f588ef91f3dec5b1e71a8785f9fa135d6e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/SSrGE.git
  tools:
  - Numpy
  - Python
  - Python 2.7
  - QuASAR
  - SSrGE
  - Scipy
  - scikit-learn
  - scikit-learn 0.18
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# snv-gene-expression-linkage

## Summary

Fit LASSO-regularized sparse linear models (SSrGE) regressing gene expression on binary SNV matrices from scRNA-seq to derive effective, expressed SNVs (eeSNVs), whose accumulated coefficients rank SNVs and genes for subpopulation identification and genotype-phenotype linkage.

## When to use

Use when you have per-cell SNV calls (e.g. .vcf-derived binary matrix) coupled with single-cell gene expression counts and need SNV-derived features that link genotype to expression phenotypes.

## When NOT to use

- Input is already a reduced feature table or the SNVs and expression come from different cells — SSrGE requires coupled DNA/RNA data from the same single cells
- The goal is de novo SNV calling from raw reads — use the SSrGE SNV_calling/download modules or QuASAR first; this skill starts from already-called SNVs
- All candidate SNV features are highly correlated — LASSO may arbitrarily select only one of highly correlated SNV features corresponding to gene expression

## Inputs

- Binary SNV matrix X (n_cells x n_SNVs) from per-cell .vcf files (one snv_filtered.vcf per cell folder)
- Gene expression matrix Y (n_cells x n_Genes) from expression files (e.g. STAR matrix_counts.txt)
- Optional CNV matrix C (n_cells x n_genes) describing CNV level per gene
- Optional snv_id_list (gene id, SNV id tuples) and gene_id_list for gene ranking

## Outputs

- Reduced eeSNV representation matrix X_r (n_cells x n_eeSNVs) via ssrge.transform(X)
- Ranked eeSNVs (rank_eeSNVs) and ranked genes (rank_genes)
- score_models and score_null_models from ssrge.score(X, Y)
- Subgroup-specific ranked/significant eeSNVs and genes via rank_features_for_a_subgroup

## How to apply

Encode the SNV matrix X as binary presence/absence per cell (n_cells x n_SNVs; X_c,n = 1 if SNV s is present in cell c, optionally 1 for 1/1 and 0.5 for 0/1 genotypes) and the expression matrix Y as n_cells x n_Genes. Instantiate SSrGE with the default LASSO model and a regularization parameter alpha (e.g. 0.01), then call fit(X, Y), optionally passing a CNV matrix C (n_cells x n_genes) as a third argument. Score fitted models against nulls with ssrge.score(X, Y), reduce X to the eeSNV representation with ssrge.transform(X), and rank eeSNVs with ssrge.rank_eeSNVs(); alternatively fix the number of eeSNVs via nb_ranked_features instead of relying on alpha. Validate the regularization choice with LinearCrossVal.regularization_path('alpha', [0.01, 0.1, 0.2]), and rank genes by instantiating SSrGE with snv_id_list (tuples of harboring gene id and SNV id, e.g. genome position) and gene_id_list. The LASSO penalization serves as feature selection to minimize spurious (false positive) SNVs, and an accumulative ranking approach selects eeSNVs linked to the expression of a particular gene.

## Related tools

- **SSrGE** (Python package that fits per-gene LASSO sparse linear models, transforms the SNV matrix into eeSNV features, and performs eeSNV/gene rankings) — https://github.com/lanagarmire/SSrGE.git
- **scikit-learn 0.18** (Underlying library providing the LASSO (and optional ElasticNet) linear models used by SSrGE)
- **QuASAR** (Upstream R package for genotyping from sequencing reads and inferring allelic imbalance at heterozygous sites, feeding SNV calls into the linkage stage) — piquelab/QuASAR
- **Python 2.7** (Runtime environment (with Numpy, Scipy, tabulate) in which the garmire_SSrGE package is installed and executed)

## Examples

```
from garmire_SSrGE.ssrge import SSrGE; ssrge = SSrGE(alpha=0.01); ssrge.fit(X, Y); X_r = ssrge.transform(X); ranked_feature = ssrge.rank_eeSNVs()
```

## Evaluation signals

- score_models from ssrge.score(X, Y) should outperform score_null_models, indicating the fitted sparse models capture real SNV-expression association
- X_r.shape from ssrge.transform(X) should be (n_cells, number of selected eeSNVs), matching alpha- or nb_ranked_features-implied dimensionality (e.g. (100, 2) with nb_ranked_features=2)
- In all datasets tested in the study, eeSNVs achieved better accuracies than gene expression for identifying subpopulations — a successful run should reproduce this accuracy advantage on held-out clustering
- Previously validated cancer-relevant genes (e.g. B2M, a mutational hotspot in Ting and Patel datasets) should appear highly ranked among top-scoring eeSNV-harboring genes
- Regularization path over alpha values [0.01, 0.1, 0.2] from LinearCrossVal should confirm the chosen alpha yields stable, non-degenerate coefficient sets

## Limitations

- SSrGE may miss some driver mutations due to incomplete DNA coverage of scRNA-seq reads
- In case of correlated features, LASSO may arbitrarily select one of highly correlated SNV features corresponding to gene expression
- eeSNVs of HLA genes as cellular state fingerprints, and eeSNVs reflecting aberrant splicing, remain speculative and unverified experimentally

## Evidence

- [abstract] We develop a linear modeling framework, SSrGE, to link eeSNVs associated with gene expression.: "We develop a linear modeling framework, SSrGE, to link eeSNVs associated with gene expression."
- [discussion] LASSO penalization as feature selection: "Secondly, LASSO penalization works as a feature selection method and minimizes the spurious SNVs (false positive) from the filtered set of eeSNVs."
- [readme] SSrGE usage and fitting: "This procedure aims to fit sparse linear models using a binary matrix (n_samples x n_SNV) as features matrix and a gene expression matrix (n_genes x n_samples) as response."
- [readme] Encoding scheme: "if a given snv (s) is present into a given cell (c), then X_c,n = 1"
- [methods] Cross-validation of alpha: "path = cross_val.regularization_path('alpha',  [0.01, 0.1, 0.2])"
- [discussion] Missing driver mutations: "Nevertheless, SSrGE may miss some driver mutations due to the incomplete DNA coverage of scRNA-seq reads."
