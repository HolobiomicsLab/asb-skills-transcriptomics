---
name: zinb-model-estimation-single-cell
description: Use when you have a raw scRNA-seq count matrix (cells x genes) and need
  a low-dimensional embedding that models zero inflation and over-dispersion, e.g.
  for clustering, t-SNE input, or batch-effect removal.
when_to_use_negative:
- Data will be analyzed with ZIFA-style Gaussian factor analysis — the article notes
  that log-transform-based models 'do not seem to be ideal for zero-inflated count
  data'
- The goal is supervised differential expression — this capability was not yet available
  and only envisioned for a future zinbwave version
- A preliminary count normalization step must precede modeling — ZINB-WaVE with the
  gene-level intercept is designed to model raw counts directly
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: zinbwave
  role: R/Bioconductor package implementing the ZINB-WaVE zero-inflated negative binomial
    model; fits the model and simulates data via zinbModel/zinbSim
  repo: https://github.com/drisso/zinbwave
- name: scRNAseq
  role: Bioconductor package used to load single-cell count matrices (e.g. S1/CA1
    data) as model input
- name: BiocManager
  role: Installer for the zinbwave Bioconductor package (requires R >=3.4, Bioconductor
    >=3.6)
- name: zinb_analysis
  role: Repository with data analysis and simulation code to reproduce the ZINB-WaVE
    paper's results and figures
  repo: https://github.com/drisso/zinb_analysis
provenance:
  source_task_ids:
  - task_007
  - task_003
  source_papers:
  - doi: 10.1038/s41467-017-02554-5
    title: A general and flexible method for signal extraction from single-cell RNA-seq
      data
  source_content_hash: sha256:1a25f26faaeb117735befe342697ed4a48c9b1fffea44a2868f1b805b80287c9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/drisso/zinbwave
  tools:
  - BiocManager
  - Bioconductor
  - R
  - scRNAseq
  - zinb_analysis
  - zinbwave
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# zinb-model-estimation-single-cell

## Summary

Fits a zero-inflated negative binomial model (ZINB-WaVE) with latent factors to single-cell RNA-seq count matrices, extracting a low-dimensional representation that accounts for dropouts, over-dispersion, and the count nature of the data without prior normalization.

## When to use

Use when you have a raw scRNA-seq count matrix (cells x genes) and need a low-dimensional embedding that models zero inflation and over-dispersion, e.g. for clustering, t-SNE input, or batch-effect removal.

## When NOT to use

- Data will be analyzed with ZIFA-style Gaussian factor analysis — the article notes that log-transform-based models 'do not seem to be ideal for zero-inflated count data'
- The goal is supervised differential expression — this capability was not yet available and only envisioned for a future zinbwave version
- A preliminary count normalization step must precede modeling — ZINB-WaVE with the gene-level intercept is designed to model raw counts directly

## Inputs

- Raw scRNA-seq count matrix (cells x genes), e.g. from the scRNAseq Bioconductor package
- Sample-level covariate matrix X (e.g. intercept 1n or batch indicators)
- Gene-level covariate matrix V (e.g. a column of ones 1J)
- Number of latent factors K
- Cell cluster labels or batch annotations for evaluating the extracted representation

## Outputs

- Estimated latent-factor matrix W (n cells x K) giving a low-dimensional representation
- Estimated model coefficients including ln(mu) and pi per gene/cell
- Simulated count datasets (via zinbModel/zinbSim) with known true parameters
- Silhouette widths for biological clusters and batches in the ZINB-WaVE embedding
- Bias and MSE tables and boxplots (e.g. bias_mse_boxplots.png) of ln(mu) and pi estimates

## How to apply

Load the raw count matrix (e.g. via the Bioconductor scRNAseq package for the S1/CA1 or Kolodziejczyk mESC datasets) and fit ZINB-WaVE with zinbwave, specifying the sample-level design matrix X (at minimum an intercept 1n, which acts as a global-scaling normalization factor when the gene-level covariate matrix V contains a column of ones 1J), the gene-level matrix V, the number of latent factors K (e.g. K=2), and genewise dispersion estimation. Decide whether unwanted variation should be regressed out: include batch or QC covariates in X so the estimated W factors capture only wanted variation. The model can be refit on simulated data generated with zinbModel/zinbSim to study estimator properties such as bias and MSE of ln(mu) and pi under misspecified K. Compute pairwise distances in the inferred low-dimensional space for downstream t-SNE, clustering, and silhouette-width evaluation.

## Related tools

- **zinbwave** (R/Bioconductor package implementing the ZINB-WaVE zero-inflated negative binomial model; fits the model and simulates data via zinbModel/zinbSim) — https://github.com/drisso/zinbwave
- **scRNAseq** (Bioconductor package used to load single-cell count matrices (e.g. S1/CA1 data) as model input)
- **BiocManager** (Installer for the zinbwave Bioconductor package (requires R >=3.4, Bioconductor >=3.6))
- **zinb_analysis** (Repository with data analysis and simulation code to reproduce the ZINB-WaVE paper's results and figures) — https://github.com/drisso/zinb_analysis

## Examples

```
BiocManager::install("zinbwave"); library(zinbwave); zinb <- zinbwave(Y, X=model.matrix(~1), V=cbind(rep(1,J)), K=2)
```

## Evaluation signals

- Under gene-level intercept inclusion, bias of ln(mu) and pi estimates stays at or very near zero when K is misspecified (K != true K), with small MSE
- Average silhouette width of biological clusters is larger when unwanted covariates (e.g. batch) are included in X, while batch silhouette is much larger in the intercept-only fit
- The low-dimensional representation should be more stable and accurate than PCA and ZIFA baselines without a preliminary normalization step
- Fits complete with a specified K and genewise dispersion; runtime should scale approximately linearly in cells and genes and approximately quadratically in K
- The estimated W factors capture wanted variation: correlated unwanted variation signals appear only if technical effects are left unaccounted for in X or V

## Limitations

- Optimization is of a non-convex likelihood, imposing a computational cost relative to PCA
- If unwanted technical variation is not accounted for in the model or prior normalization, the low-rank matrix W will capture confounding effects
- There is no guarantee that the low-dimensional signal extracted by ZINB-WaVE is biologically relevant
- Exactly reproducing the paper's analyses requires version 0.1.1 of the zinbwave package
- No compelling examples were found where adding a gene-level covariate improves signal extraction

## Evidence

- [abstract] ZINB-WaVE is a flexible zero-inflated NB model for scRNA-seq signal extraction: "we present a general and flexible zero-inflated negative binomial model (ZINB-WaVE)"
- [intro] The method is implemented in the zinbwave R/Bioconductor package: "The approach is implemented in the open-source R package zinbwave, publicly available through the Bioconductor Project"
- [discussion] Including a gene-level column of ones makes the cell-level intercept a global-scaling factor so raw counts can be modeled: "the corresponding cell-level intercept acts as a global-scaling normalization factor"
- [discussion] ZINB-WaVE estimators are robust and unbiased for biological signals: "We have shown with simulated and real data analyses that ZINB-WaVE leads to robust and unbiased estimators of the underlying biological signals."
- [methods] Installation of the zinbwave package: "BiocManager::install("zinbwave")"
- [readme] Reproducing the paper's analyses requires a specific zinbwave version: [evidence span withheld — anchor status: external_doc]
