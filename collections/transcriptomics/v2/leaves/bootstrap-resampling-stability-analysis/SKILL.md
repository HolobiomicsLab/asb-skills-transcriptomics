---
name: bootstrap-resampling-stability-analysis
description: Use when comparing feature representations (e.g., eeSNV vs gene expression
  matrices) for subpopulation identification in scRNA-seq and you need stability of
  clustering accuracy quantified across resampled runs rather than a single clustering
  outcome.
when_to_use_negative:
- You have only a single clustering run and no ground-truth subpopulation labels —
  AMI against known labels cannot be computed
- You only need to rank eeSNVs or genes (use ssrge.rank_eeSNVs() / ssrge.rank_genes())
  rather than compare clustering stability across feature types
- The input is already a stability/agreement summary table; no further bootstrap resampling
  is warranted
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0203
tools:
- name: scikit-learn 0.18
  role: Implements PCA, factor analysis, K-means, and agglomerative clustering (WARD
    linkage) used in each bootstrap run
- name: SSrGE
  role: Produces the eeSNV feature matrices (via ssrge.fit and ssrge.transform) that
    are benchmarked against gene expression in the bootstrap comparison
  repo: https://github.com/lanagarmire/SSrGE.git
- name: Numpy
  role: Array computation underlying the SNV/GE matrices and bootstrap resampling
- name: Scipy
  role: Numerical routines supporting the Python analysis stack
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-018-07170-5
    title: Using single nucleotide variations in single-cell RNA-seq to identify subpopulations
      and genotype-phenotype linkage
  source_content_hash: sha256:90b7723af8d6c29840e50c8070ffbee6fe716e7601021356b422fcd72ab900da
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
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# bootstrap-resampling-stability-analysis

## Summary

Assess clustering/feature robustness by repeating the full clustering pipeline (e.g., PCA/FA + K-means, agglomerative WARD, or SIMLR) over ~30 bootstrap resampling runs and computing adjusted mutual information (AMI, bounded [0,1]) against known labels, then reporting mean and variability per condition.

## When to use

Use when comparing feature representations (e.g., eeSNV vs gene expression matrices) for subpopulation identification in scRNA-seq and you need stability of clustering accuracy quantified across resampled runs rather than a single clustering outcome.

## When NOT to use

- You have only a single clustering run and no ground-truth subpopulation labels — AMI against known labels cannot be computed
- You only need to rank eeSNVs or genes (use ssrge.rank_eeSNVs() / ssrge.rank_genes()) rather than compare clustering stability across feature types
- The input is already a stability/agreement summary table; no further bootstrap resampling is warranted

## Inputs

- Feature matrices per dataset: eeSNV feature matrix (n_cells x n_ranked_eeSNVs) from ssrge.transform(X) and full gene expression matrix (n_cells x n_Genes)
- Known subpopulation labels for each cell in each of the five GEO scRNA-seq datasets (GSE73121, GSE51372, GSE67980, GSE57872, GSE75688)
- Clustering configuration: PCA/factor-analysis reduction + K-means, PCA/FA + agglomerative WARD, and SIMLR

## Outputs

- Per-run AMI score in [0, 1] for each dataset x feature type x clustering strategy x bootstrap run
- Aggregated mean and variability of AMI per dataset, feature type, and clustering strategy
- Full comparison table written to ami_comparison_results.csv

## How to apply

For each dataset and each feature type (here, eeSNV-transformed matrices from ssrge.transform(X) versus full GE matrices), run the clustering strategies under comparison — PCA or factor-analysis dimensionality reduction followed by K-means, PCA/FA followed by agglomerative clustering with WARD linkage, and SIMLR — implemented in scikit-learn. Repeat each clustering over 30 bootstrap resampling runs, and for every run compute adjusted mutual information (AMI) against the known subpopulation labels, which bounds scores in [0, 1]. Aggregate the mean and variability of AMI per dataset, feature type, and clustering strategy, and write the full comparison to a results table (e.g., ami_comparison_results.csv). A successful application reproduces the expected Fig. 2 pattern: the stronger feature set (eeSNVs) achieves higher AMI than gene expression in all datasets tested, consistently across all five clustering approaches, demonstrating the result is not an artifact of one algorithm or one resample.

## Related tools

- **scikit-learn 0.18** (Implements PCA, factor analysis, K-means, and agglomerative clustering (WARD linkage) used in each bootstrap run)
- **SSrGE** (Produces the eeSNV feature matrices (via ssrge.fit and ssrge.transform) that are benchmarked against gene expression in the bootstrap comparison) — https://github.com/lanagarmire/SSrGE.git
- **Numpy** (Array computation underlying the SNV/GE matrices and bootstrap resampling)
- **Scipy** (Numerical routines supporting the Python analysis stack)

## Examples

```
from garmire_SSrGE.ssrge import SSrGE; ssrge = SSrGE(); ssrge.fit(X, Y); X_r = ssrge.transform(X)
```

## Evaluation signals

- AMI values for every run are bounded in [0, 1]
- Aggregated table (ami_comparison_results.csv) covers all 5 datasets x 2 feature types (eeSNV vs GE) x 5 clustering strategies x 30 bootstrap runs
- Mean AMI pattern matches Fig. 2: eeSNV features outperform GE for subpopulation identification in all datasets tested
- The eeSNV advantage over GE is consistent across PCA/FA + K-means, PCA/FA + agglomerative WARD, and SIMLR, indicating algorithm-independence
- Variability of AMI across the 30 runs is reported alongside means so stability, not just central tendency, is assessable

## Limitations

- Bootstrap stability results depend on availability of trusted subpopulation labels; AMI cannot be computed without them
- Gene expression features are noisy (batch effects, cell-cycle-dependent variation), which the benchmark captures, but explanations for eeSNV superiority remain speculative and not experimentally verified
- SSrGE-based eeSNV features may miss driver mutations due to incomplete DNA coverage of scRNA-seq reads, which can affect the benchmarked feature space

## Evidence

- [abstract] eeSNVs are consistently better features than GE for retrieving cancer cell subpopulations, independent of the clustering method used: "In all the datasets tested, eeSNVs achieve better accuracies than gene expression for identifying subpopulations."
- [methods] Bootstrap protocol with 30 runs and AMI metric: "ranked_feature = ssrge.rank_eeSNVs()"
- [readme] Feature matrices feeding the bootstrap comparison are produced by SSrGE transform: "SSrGE takes as input two matrices (A SNV matrix (n_cells x n_SNVs) and a Gene matrix (n_cells x n_Genes)"
- [intro] GE features are noisy, motivating resampling-based robustness assessment: "GE features are subject to a significant amount of noises4. For example, GE might be affected by batch effect"
- [readme] Cross-validation / repeated fitting machinery available in the SSrGE package: "the package allows to parallelize the fitting procedures, implements a cross-validation procedure and performs eeSNVs and gene rankings."
