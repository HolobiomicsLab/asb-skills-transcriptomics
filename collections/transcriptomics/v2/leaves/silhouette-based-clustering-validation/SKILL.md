---
name: silhouette-based-clustering-validation
description: Use when you need to quantify whether an engineered/derived cell feature
  representation (e.g., Spearman or Pearson correlation, cosine, Euclidean, autoencoder
  features) improves consistency with known cell-type labels in scRNA-seq data.
when_to_use_negative:
- No ground-truth cell-type labels are available — silhouette against true labels
  cannot be computed
- The goal is clustering accuracy itself (e.g., ARI against labels) rather than assessing
  whether a derived feature space better aligns with labels than the original expression
  profiles
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SSCC
  role: Framework whose Spearman-correlation feature construction is evaluated by
    silhouette comparison; provides the feature spaces being validated
  repo: https://github.com/Japrin/sscClust
- name: R
  role: Implementation language; silhouette computation, scatter plotting, and composite
    figure assembly are performed in R via sscClust
  repo: https://github.com/Japrin/sscClust
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.gpb.2018.10.003
    title: 'SSCC: A Novel Computational Framework for Rapid and Accurate Clustering
      Large-Scale Single Cell RNA-Seq Data'
  source_content_hash: sha256:d20dedb61b4ea1eac7bcb11c413d82f0b52ac8fe0e1227b0e89034789b0d0077
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Japrin/sscClust
  tools:
  - R
  - SSCC
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# silhouette-based-clustering-validation

## Summary

Validates scRNA-seq feature construction/extraction methods by computing silhouette coefficients of cell features against true cluster labels and comparing them per cell to those of the original expression profiles, as done in the SSCC paper's Figure 2 across five benchmark datasets.

## When to use

Use when you need to quantify whether an engineered/derived cell feature representation (e.g., Spearman or Pearson correlation, cosine, Euclidean, autoencoder features) improves consistency with known cell-type labels in scRNA-seq data.

## When NOT to use

- No ground-truth cell-type labels are available — silhouette against true labels cannot be computed
- The goal is clustering accuracy itself (e.g., ARI against labels) rather than assessing whether a derived feature space better aligns with labels than the original expression profiles

## Inputs

- scRNA-seq expression matrices from the five benchmark datasets (Kolodziejczyk, Pollen, Usoskin, Zeisel, Zheng) loadable via sscClust (e.g., sce.Pollen SingleCellExperiment object)
- True cell-type label vector for each dataset

## Outputs

- Per-cell silhouette coefficients for the original data and each of the five feature representations (Euclidean, cosine, Pearson, Spearman, autoencoder)
- Per-cell scatter panels (original vs. feature silhouette values) per dataset per feature type
- Fraction of cells above the diagonal per dataset/feature-type comparison
- Composite Figure 2-style plot saved as figure2_silhouette_scatter.png

## How to apply

Load each benchmark scRNA-seq dataset (Kolodziejczyk, Pollen, Usoskin, Zeisel, Zheng) with its true cell-type labels using the sscClust R package (github.com/Japrin/sscClust). Construct alternative feature representations per dataset — Euclidean-based, cosine-based, Pearson-correlation-based, Spearman-correlation-based, and autoencoder-derived features — alongside the original expression profiles. For each representation compute silhouette coefficients against the true labels, then for each dataset plot per-cell silhouette values of the original data versus those of each feature type and calculate the fraction of cells falling above the diagonal. A feature method is judged better when the majority of cells lie above the diagonal (higher silhouette than the original data); in the SSCC study only the Spearman correlation-based features consistently improved accordance with true labels across all five datasets.

## Related tools

- **SSCC** (Framework whose Spearman-correlation feature construction is evaluated by silhouette comparison; provides the feature spaces being validated) — https://github.com/Japrin/sscClust
- **R** (Implementation language; silhouette computation, scatter plotting, and composite figure assembly are performed in R via sscClust) — https://github.com/Japrin/sscClust

## Examples

```
library(sscClust); data("sce.Pollen"); sce <- ssc.run(sce.Pollen, subsampling=FALSE, k.batch=11)
```

## Evaluation signals

- For each dataset and feature type, the fraction of cells above the original-vs-feature silhouette diagonal is computed and reported
- A feature method 'passes' only if it consistently raises silhouette values across all five datasets — as Spearman-based features did in the paper
- The composite output matches the Figure 2 layout: per-cell scatter panels of original silhouette vs. each feature-type silhouette
- Silhouette values are computed against the same true cluster labels for both original data and every feature representation (fair pairwise comparison)

## Limitations

- Validation relies on curated true cell-type labels; only Spearman correlation-based features consistently improved accordance across all five benchmark datasets — Euclidean, cosine, Pearson, and autoencoder features did not
- Subsampling strategies (as in SCC/SSCC) can introduce biases that affect accuracy and robustness, which silhouette comparison is meant to expose
- Computational cost grows with dataset size; the README recommends R linked to an optimized BLAS library (ATLAS, MKL) for large datasets

## Evidence

- [results] Silhouette comparison workflow: [evidence span withheld — anchor status: not_found]
- [discussion] Feature construction rationale: "with Spearman correlations as the feature engineering and projecting methods, we formulate a framework named as SSCC"
- [abstract] Tool availability: "An R implementation of SSCC is available at https://github.com/Japrin/sscClust"
- [readme] Feature space description from README: "projecting data to a feature space using spearman correlation which make visualiaztion and clustering more efficient"
- [readme] Example data access from README: "data("sce.Pollen")
sce.all <- ssc.run(sce.Pollen, subsampling=F, k.batch=11)"
