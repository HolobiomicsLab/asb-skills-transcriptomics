---
name: single-cell-rna-seq-clustering-stability-analysis
description: Use when you have raw vs. imputed scRNA-seq expression matrices with
  known cell-type labels and must quantify whether imputation improves or distorts
  cell clustering quality and gene-level dispersion.
when_to_use_negative:
- The input has no known cell-type labels, since k-means must be run with the true
  number of cell types k and the ARI/PI requires ground-truth labels
- You only have a single unperturbed run and want a stability claim — the skill requires
  perturbing each dataset to assess transcriptome-landscape preservation
- The data are bulk RNA-seq or already a clustering result/feature table rather than
  a cell-by-gene expression matrix
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: scISR
  role: Imputation method under test; its output matrix is clustered and compared
    against raw data and competitor methods for PI and intra-dispersion
  repo: https://github.com/duct317/scISR
- name: R stats
  role: Provides the lm function used in scISR's subspace regression and statistical
    testing (Wilcoxon signed-rank) for the PI comparison
- name: scImpute
  role: Competitor imputation method whose output PI and intra-dispersion are compared
    against scISR
- name: SAVER
  role: Competitor imputation method included in the benchmark comparison
- name: MAGIC
  role: Competitor imputation method included in the benchmark comparison
- name: DrImpute
  role: Competitor imputation method included in the benchmark comparison
- name: scScope
  role: Competitor imputation method included in the benchmark comparison
- name: DCA
  role: Competitor imputation method included in the benchmark comparison
- name: DeepImpute
  role: Competitor imputation method included in the benchmark comparison
- name: BISCUIT
  role: Competitor imputation method included in the benchmark comparison
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-022-06500-4
    title: A novel method for single-cell data imputation using subspace regression
  source_content_hash: sha256:212f01134cf3d8c897cb3ec7f9393f3ce57423a5f275373faa4382392311d175
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duct317/scISR
  tools:
  - BISCUIT
  - DCA
  - DeepImpute
  - DrImpute
  - MAGIC
  - R stats
  - SAVER
  - scISR
  - scImpute
  - scScope
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# single-cell-rna-seq-clustering-stability-analysis

## Summary

Evaluates whether single-cell imputation (e.g. scISR) genuinely preserves the transcriptome landscape by comparing clustering quality (Purity Index, ARI-based, via k-means at the true k) between raw and imputed expression matrices, plus a median gene-level normalized intra-dispersion check.

## When to use

Use when you have raw vs. imputed scRNA-seq expression matrices with known cell-type labels and must quantify whether imputation improves or distorts cell clustering quality and gene-level dispersion.

## When NOT to use

- The input has no known cell-type labels, since k-means must be run with the true number of cell types k and the ARI/PI requires ground-truth labels
- You only have a single unperturbed run and want a stability claim — the skill requires perturbing each dataset to assess transcriptome-landscape preservation
- The data are bulk RNA-seq or already a clustering result/feature table rather than a cell-by-gene expression matrix

## Inputs

- Raw scRNA-seq expression matrix (e.g. FPKM/RPKM/CPM/TPM/RPM counts) per dataset, e.g. Fan GSE53386, Treutlein GSE52583, Yan GSE36552, Goolam E-MTAB-3321, Deng GSE45719, Pollen SRP041736, Darmanis GSE67835, Usoskin GSE59739, Camp GSE75140, Klein GSE65525
- Imputed expression matrices from scISR and competitor methods (scImpute, SAVER, MAGIC, DrImpute, scScope, DCA, DeepImpute, BISCUIT)
- True cell-type labels per dataset (used to set k and compute ARI/PI)

## Outputs

- Per-dataset Purity Index (ARI-based) values for raw data and each imputation method across 25 datasets
- Wilcoxon signed-rank test result comparing scISR PI vs raw (reference: 15/25 wins, p = 0.007)
- Median gene-level normalized intra dispersion per method (reference: 3.6e-3 for scISR)
- Aggregated stability_summary.csv table

## How to apply

For each dataset, rescale values with log2 when the value range exceeds 100, and pre-filter genes expressed in fewer than two cells and genes with less than one percent non-zero entries. Reduce dimensionality (e.g. PCA, prcomp_irlba n=50 in the README) and cluster cells with k-means using the true number of cell types k as the number of clusters (nstart=2000, iter.max=2000, fixed seed), then compute the clustering-quality index (PI/ARI) against known labels for raw data and each imputed matrix. Perturb each dataset and repeat to obtain per-dataset PI values for raw, scISR, and competitors (scImpute, SAVER, MAGIC, DrImpute, scScope, DCA, DeepImpute, BISCUIT); count datasets where scISR's PI exceeds raw and assess with a one-sided Wilcoxon signed-rank test. Also compute the median gene-level normalized intra dispersion of the imputed data to check that biological cell-to-cell variation is not over-smoothed. Assemble per-dataset PI and dispersion values into stability_summary.csv; the reference outcome is scISR above raw in 15/25 datasets (the only method averaging above raw), Wilcoxon p = 0.007, and median gene-level normalized intra dispersion of 3.6e-3.

## Related tools

- **scISR** (Imputation method under test; its output matrix is clustered and compared against raw data and competitor methods for PI and intra-dispersion) — https://github.com/duct317/scISR
- **R stats** (Provides the lm function used in scISR's subspace regression and statistical testing (Wilcoxon signed-rank) for the PI comparison)
- **scImpute** (Competitor imputation method whose output PI and intra-dispersion are compared against scISR)
- **SAVER** (Competitor imputation method included in the benchmark comparison)
- **MAGIC** (Competitor imputation method included in the benchmark comparison)
- **DrImpute** (Competitor imputation method included in the benchmark comparison)
- **scScope** (Competitor imputation method included in the benchmark comparison)
- **DCA** (Competitor imputation method included in the benchmark comparison)
- **DeepImpute** (Competitor imputation method included in the benchmark comparison)
- **BISCUIT** (Competitor imputation method included in the benchmark comparison)

## Examples

```
imputed <- scISR(data = raw); pca <- irlba::prcomp_irlba(t(imputed), n = 50)$x; cluster <- kmeans(pca, length(unique(label)), nstart = 2000, iter.max = 2000)$cluster; adjustedRandIndex(cluster, label)
```

## Evaluation signals

- scISR's PI exceeds raw data's PI in 15 out of 25 datasets
- One-sided Wilcoxon signed-rank test over PI differences vs raw and all competitors yields p = 0.007
- Median gene-level normalized intra dispersion for scISR equals 3.6e-3, comparable to raw and better than competitors
- ARI is computed by k-means with nstart=2000, iter.max=2000, fixed seed, and PCA-reduced (n=50) data, and must exceed the raw-data ARI on most datasets (e.g. Goolam in the README example)
- Output table stability_summary.csv contains per-dataset PI rows for raw, scISR, and all 8 competitor methods

## Limitations

- The R stats package and scISR software versions are not stated, limiting reproducibility
- The article defers key robustness evidence to supplementary figures (S19, S21, S24, S5) not included here
- The repository has no changelog ('_No changelog found._'), limiting reproducibility of repository content

## Evidence

- [results] we use k-means to cluster the cells using the true number of cell types k as the number of clusters.: "we use k-means to cluster the cells using the true number of cell types k as the number of clusters."
- [abstract] evaluation on 25 public datasets: "Our comprehensive evaluation using 25 publicly available scRNA-seq datasets and various simulation scenarios against five state-of-the-art methods demonstrates that scISR is better than other"
- [results] gene filtering thresholds: "(i) genes expressed in less than two cells; and (ii) genes that have less than one percent of non-zero-valued entries."
- [results] log2 rescaling rule: "For datasets with the range of values larger than 100, we rescale the data using log transformation (base 2)."
- [readme] README clustering assessment protocol: "pca_imputed <- irlba::prcomp_irlba(t(imputed), n = 50)$x"
- [intro] over-smoothing risk: "tending to over-smoothen and remove the cell-to-cell stochasticity that represents meaningful biological variations in gene expression."
