---
name: autoencoder-feature-extraction-scrna
description: Use when benchmarking autoencoder-derived latent features of scRNA-seq
  data against original profiles and correlation-based features (Euclidean, cosine,
  Pearson, Spearman) via silhouette values against true cell-type labels.
when_to_use_negative:
- You need a feature construction method guaranteed to improve accordance with true
  cluster labels — the paper found only the Spearman correlation-based method consistently
  improved across all five datasets, not the autoencoder.
- The input is already a low-dimensional feature/embedding table rather than scRNA-seq
  expression profiles that need encoding.
- Working with a very large dataset without an R build linked to an optimized BLAS
  (ATLAS, MKL) — the sscClust README warns calculations become expensive and recommends
  Microsoft R Open on Windows.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SSCC
  role: Clustering framework whose feature-construction benchmark includes autoencoder-derived
    features, evaluated by silhouette values against true labels on five datasets
  repo: https://github.com/Japrin/sscClust
- name: R
  role: Implementation environment; sscClust is an R package providing the ssc.run
    pipeline and example SingleCellExperiment data (e.g., sce.Pollen)
  repo: https://github.com/Japrin/sscClust
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.gpb.2018.10.003
    title: 'SSCC: A Novel Computational Framework for Rapid and Accurate Clustering
      Large-Scale Single Cell RNA-Seq Data'
  source_content_hash: sha256:c8b0cd486fad9f775bf5a98e55a3df378c4f5188d7620c7ba429718449dfa575
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Japrin/sscClust
  tools:
  - R
  - SSCC
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# autoencoder-feature-extraction-scrna

## Summary

Trains an autoencoder on scRNA-seq expression profiles to derive per-cell latent features and scores them by silhouette coefficients against true cell-type labels, as one of the five feature-construction variants (Euclidean, cosine, Pearson, Spearman, autoencoder) benchmarked in the SSCC framework (Figure 2).

## When to use

Use when benchmarking autoencoder-derived latent features of scRNA-seq data against original profiles and correlation-based features (Euclidean, cosine, Pearson, Spearman) via silhouette values against true cell-type labels.

## When NOT to use

- You need a feature construction method guaranteed to improve accordance with true cluster labels — the paper found only the Spearman correlation-based method consistently improved across all five datasets, not the autoencoder.
- The input is already a low-dimensional feature/embedding table rather than scRNA-seq expression profiles that need encoding.
- Working with a very large dataset without an R build linked to an optimized BLAS (ATLAS, MKL) — the sscClust README warns calculations become expensive and recommends Microsoft R Open on Windows.

## Inputs

- scRNA-seq expression matrix with true cell-type labels as a SingleCellExperiment object (e.g., sce.Pollen bundled with sscClust)
- The five benchmark scRNA-seq datasets: Kolodziejczyk, Pollen, Usoskin, Zeisel, Zheng
- Original expression profiles from which the autoencoder latent code is learned

## Outputs

- Per-cell autoencoder latent feature matrix
- Per-cell silhouette coefficients against the true cluster labels for the original data and each feature representation
- Scatter panels of original vs. feature silhouette values with the fraction of cells above the diagonal, assembled as figure2_silhouette_scatter.png

## How to apply

Load each benchmark dataset (Kolodziejczyk, Pollen, Usoskin, Zeisel, Zheng) with its true cell-type labels through the sscClust R package (e.g., data('sce.Pollen')). Train an autoencoder on the original expression profiles to obtain per-cell latent features, building this representation alongside the Euclidean-, cosine-, Pearson-, and Spearman-correlation-based features. Compute silhouette coefficients of every cell against the true labels for each feature representation and for the original data. For each dataset, scatter original-data silhouette values versus autoencoder-feature silhouette values, draw the y=x diagonal, and compute the fraction of cells above the diagonal as the improvement measure. Assemble the per-cell panels into the Figure 2 composite and save as figure2_silhouette_scatter.png; expect the autoencoder features not to lie consistently above the diagonal — per the paper, only the Spearman correlation-based features improved accordance with true labels on all five datasets.

## Related tools

- **SSCC** (Clustering framework whose feature-construction benchmark includes autoencoder-derived features, evaluated by silhouette values against true labels on five datasets) — https://github.com/Japrin/sscClust
- **R** (Implementation environment; sscClust is an R package providing the ssc.run pipeline and example SingleCellExperiment data (e.g., sce.Pollen)) — https://github.com/Japrin/sscClust

## Examples

```
data("sce.Pollen"); sce.all <- ssc.run(sce.Pollen, subsampling=F, k.batch=11)
```

## Evaluation signals

- Silhouette coefficients are computed per cell against the same true cell-type labels used for the original data, on all five benchmark datasets (Kolodziejczyk, Pollen, Usoskin, Zeisel, Zheng).
- Each scatter panel plots original-data silhouette values against feature silhouette values with the y=x diagonal, and the fraction of cells above the diagonal is reported as the improvement statistic.
- Consistency check with the paper: the autoencoder fraction above the diagonal should not exceed ~0.5 on all five datasets simultaneously — only Spearman achieved that.
- The composite figure reproduces the Figure 2 panel layout and is saved as figure2_silhouette_scatter.png.
- Pipeline sanity check: data('sce.Pollen') loads in R and ssc.run(...) with k.batch=11 completes on the reduced data.

## Limitations

- Autoencoder-derived features did not consistently improve the accordance between cell features and true cluster labels; only the Spearman correlation-based method did so across all five benchmark datasets.
- Computation becomes expensive as datasets grow; the sscClust README recommends R linked to an optimized BLAS library (ATLAS, MKL) or Microsoft R Open for Windows users.
- sscClust requires the companion sscVis package to be installed first (devtools::install_github('Japrin/sscVis')).

## Evidence

- [abstract] SSCC feature-construction framework: "Here we proposed Spearman subsampling-clustering-classification (SSCC), a new clustering framework based on random projection and feature construction"
- [discussion] Feature engineering via Spearman correlation: "with Spearman correlations as the feature engineering and projecting methods, we formulate a framework named as SSCC"
- [readme] Feature-space projection in sscClust: "projecting data to a feature space using spearman correlation which make visualiaztion and clustering more efficient"
- [readme] Computational cost caveat: "Calculation will become expensive when the dataset become large, so we stongly recommand use R which linked to optimized BLAS library, such as ATLAS, MKL etc."
- [readme] ssc.run example call: [evidence span withheld — anchor status: external_doc]
