---
name: imputation-method-benchmarking
description: Use when comparing whether an imputation method improves downstream clustering
  quality over raw data and competitor imputers on real scRNA-seq datasets, using
  clustering-quality indices and significance tests.
when_to_use_negative:
- The goal is imputation accuracy on masked entries rather than downstream clustering
  quality (use the 30%-masked-genes simulation protocol instead)
- The dataset lacks known cell-type labels, since k requires the true number of cell
  types and ARI-based PI requires reference labels
- Input is bulk RNA-seq or non-single-cell data, which cannot measure cell-to-cell
  heterogeneity as described
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3308
tools:
- name: scISR
  role: Imputation method under evaluation; produces imputed matrices from raw scRNA-seq
    data via hyper-geometric dropout testing and subspace regression
  repo: https://github.com/duct317/scISR
- name: scImpute
  role: Competitor model-based imputation method benchmarked against scISR on the
    same datasets
- name: SAVER
  role: Competitor Poisson-Gamma Bayesian imputation method benchmarked against scISR
- name: MAGIC
  role: Competitor heat-diffusion imputation method benchmarked against scISR
- name: DrImpute
  role: Competitor cluster-ensemble imputation method benchmarked against scISR
- name: scScope
  role: Competitor recurrent-network imputation method benchmarked against scISR
- name: DCA
  role: Competitor autoencoder-based imputation method benchmarked against scISR
- name: DeepImpute
  role: Competitor sub-neural-network imputation method benchmarked against scISR
- name: BISCUIT
  role: Competitor Dirichlet-process-mixture imputation method benchmarked against
    scISR
- name: R stats
  role: Used in the scISR implementation (lm function) and for the Wilcoxon signed-rank
    significance testing
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-022-06500-4
    title: A novel method for single-cell data imputation using subspace regression
  source_content_hash: sha256:c451490d8d54b6ee9719c79f6e6bf373debe2d0e9560723c08aa0b1a4fea1605
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# imputation-method-benchmarking

## Summary

Benchmark scRNA-seq imputation methods (scISR, scImpute, SAVER, MAGIC, DrImpute, scScope, DCA, DeepImpute, BISCUIT) against raw data by clustering imputed matrices with k-means and scoring cluster quality, following the scISR evaluation protocol.

## When to use

Use when comparing whether an imputation method improves downstream clustering quality over raw data and competitor imputers on real scRNA-seq datasets, using clustering-quality indices and significance tests.

## When NOT to use

- The goal is imputation accuracy on masked entries rather than downstream clustering quality (use the 30%-masked-genes simulation protocol instead)
- The dataset lacks known cell-type labels, since k requires the true number of cell types and ARI-based PI requires reference labels
- Input is bulk RNA-seq or non-single-cell data, which cannot measure cell-to-cell heterogeneity as described

## Inputs

- Raw scRNA-seq count/expression matrices with known cell-type labels (FPKM/RPKM/TPM/CPM/RPM/UMI formats)
- Imputed expression matrices produced by scISR, scImpute, SAVER, MAGIC, DrImpute, scScope, DCA, DeepImpute, BISCUIT
- True number of cell types k per dataset

## Outputs

- Per-dataset clustering-quality indices (Purity Index / ARI) for raw and each imputation method
- Wilcoxon signed-rank test statistic and p-value comparing each method's PI to raw data
- Median gene-level normalized intra dispersion per method
- Aggregated summary table (stability_summary.csv)

## How to apply

Load raw and imputed expression matrices for the public scRNA-seq datasets (e.g. Fan GSE53386, Treutlein GSE52583, Yan GSE36552, Goolam E-MTAB-3321, Deng GSE45719, Pollen SRP041736, Darmanis GSE67835, Usoskin GSE59739, Camp GSE75140, Klein GSE65525). Preprocess by removing genes expressed in fewer than two cells and genes with less than one percent non-zero entries, and rescale data with a base-2 log transformation when the value range exceeds 100. Perturb each dataset, then cluster cells with k-means using the true number of cell types k as the number of clusters, computing the clustering-quality index (Purity Index, ARI-based) for raw data, scISR, and each competitor. Count datasets where scISR's PI exceeds the raw data's and assess the difference with a one-sided Wilcoxon signed-rank test; optionally compute median gene-level normalized intra dispersion. Assemble aggregated values into a summary table (e.g. stability_summary.csv) for inspection.

## Related tools

- **scISR** (Imputation method under evaluation; produces imputed matrices from raw scRNA-seq data via hyper-geometric dropout testing and subspace regression) — https://github.com/duct317/scISR
- **scImpute** (Competitor model-based imputation method benchmarked against scISR on the same datasets)
- **SAVER** (Competitor Poisson-Gamma Bayesian imputation method benchmarked against scISR)
- **MAGIC** (Competitor heat-diffusion imputation method benchmarked against scISR)
- **DrImpute** (Competitor cluster-ensemble imputation method benchmarked against scISR)
- **scScope** (Competitor recurrent-network imputation method benchmarked against scISR)
- **DCA** (Competitor autoencoder-based imputation method benchmarked against scISR)
- **DeepImpute** (Competitor sub-neural-network imputation method benchmarked against scISR)
- **BISCUIT** (Competitor Dirichlet-process-mixture imputation method benchmarked against scISR)
- **R stats** (Used in the scISR implementation (lm function) and for the Wilcoxon signed-rank significance testing)

## Examples

```
Rscript -e "library(scISR); data('Goolam'); imputed <- scISR(data=Goolam$data); library(irlba); pca <- irlba::prcomp_irlba(t(imputed), n=50)$x; cl <- kmeans(pca, length(unique(Goolam$label)), nstart=2000, iter.max=2000)$cluster; print(round(adjustedRandIndex(cl, Goolam$label),3))"
```

## Evaluation signals

- scISR's PI exceeds raw data's PI in 15 out of 25 datasets and is the only method whose average PI exceeds raw data's
- One-sided Wilcoxon signed-rank test over raw data and all competitor methods yields p = 0.007 in scISR's favor
- Median gene-level normalized intra dispersion for scISR is 3.6e-3 versus raw data and all competitors
- k-means is run with exactly k = the true number of cell types and clustering quality is scored as ARI against known labels
- Output summary table is written to stability_summary.csv with aggregated per-method values

## Limitations

- The 25 real scRNA-seq datasets are not exhaustively named in some sections and no accession IDs are given there, limiting reproducibility
- No version stated for the R stats package or the scISR software
- Simulated dataset generation parameters only partially specified (illustrative example only), limiting reproducibility of the 116 simulated datasets
- Evaluation depends on datasets with known cell-type labels to set k and compute ARI-based PI

## Evidence

- [methods] k-means clustering with true number of cell types k: "we use k-means to cluster the cells using the true number of cell types k as the number of clusters."
- [methods] Filtering and log2 rescaling rules: "(i) genes expressed in less than two cells; and (ii) genes that have less than one percent of non-zero-valued entries."
- [abstract] Evaluation scope on 25 datasets: "Our comprehensive evaluation using 25 publicly available scRNA-seq datasets and various simulation scenarios against five state-of-the-art methods demonstrates that scISR is better than other"
- [intro] Benchmarked imputation methods: "Methods in this category include MAGIC20, DrImpute21, scScope22, DCA23, and DeepImpute24. MAGIC imputes zero values using heat diffusion25."
- [readme] README example invocation for imputation and ARI assessment: "print(paste('ARI of clusters using imputed data:', round(adjustedRandIndex(cluster_imputed, label),3)))"
