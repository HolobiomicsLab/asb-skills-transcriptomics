---
name: tweedie-semicontinuous-count-modeling
description: Use when the input is a sparse, semi-continuous omics feature-by-sample
  matrix (counts or relative abundances) with point mass at zero and heavy tails,
  and you need per-feature differential testing between conditions.
when_to_use_negative:
- When zero-inflation is a property specific to only a subset of genes/features —
  Tweedieverse is not architected for that situation
- When the analysis requires multi-subject designs, individual-level differential
  expression, or differential subpopulation analysis — these extensions are not yet
  validated
- When the input is already a normalized/transformed expression matrix with no zero
  point mass, since the method relies on a semi-continuous response with log(S) as
  an offset rather than ad hoc normalization or transformation
edam_operation: http://edamontology.org/operation_3565
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_2269
tools:
- name: Tweedieverse
  role: Core R/Bioconductor package fitting Tweedie (CPLM/ZICP) GLMs for differential
    analysis of semi-continuous omics data
  repo: https://github.com/himelmallick/Tweedieverse
- name: tweedie (R package)
  role: Evaluates the Tweedie density function (via Fourier inversion) for profile
    likelihood computation
- name: cplm (R package)
  role: Fits the initial Compound Poisson GLM at p = 1.5 used to initialize coefficients
- name: statmod (R package)
  role: Supports the computation of Tweedie likelihood components
- name: R
  role: Runtime environment; glm() and optim() functions of the stats base R package
    used in the fitting procedure
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/sim.9430
    title: Differential expression of single‐cell RNA‐seq data using Tweedie models
  source_content_hash: sha256:31425bab30c40bc7af43fe897497a79112395314036136687d4c70d1e5e3d55d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/himelmallick/Tweedieverse
  tools:
  - R
  - Tweedieverse
  - cplm (R package)
  - statmod (R package)
  - tweedie (R package)
  edam_operation: http://edamontology.org/operation_3565
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# tweedie-semicontinuous-count-modeling

## Summary

Fits Compound Poisson Linear Models (CPLM/ZICP) with a Tweedie-distributed response (1 < p < 2) to semi-continuous, zero-inflated omics data — including scRNA-seq UMI/non-UMI counts, microbiome counts or relative abundances, and metabolomics peak intensities — for feature-level differential expression testing.

## When to use

Use when the input is a sparse, semi-continuous omics feature-by-sample matrix (counts or relative abundances) with point mass at zero and heavy tails, and you need per-feature differential testing between conditions.

## When NOT to use

- When zero-inflation is a property specific to only a subset of genes/features — Tweedieverse is not architected for that situation
- When the analysis requires multi-subject designs, individual-level differential expression, or differential subpopulation analysis — these extensions are not yet validated
- When the input is already a normalized/transformed expression matrix with no zero point mass, since the method relies on a semi-continuous response with log(S) as an offset rather than ad hoc normalization or transformation

## Inputs

- Feature-by-sample count or relative-abundance matrix (e.g., scRNA-seq UMI/non-UMI counts, microbiome taxonomic/functional profiles, metabolomics compound abundances or MS peak intensities)
- Sample metadata / binary or covariate design matrix
- Optional per-sample normalization factors / library sizes (e.g., SCRAN-estimated cell-specific sizes)

## Outputs

- Per-feature table of coefficients, Wald-test p-values, FDR-adjusted q-values, fold change, and estimated per-feature Tweedie index parameter p (e.g., de_results.csv)

## How to apply

Load the feature-by-sample abundance matrix and metadata, compute per-sample library size (SCRAN for scRNA-seq) and set up a design matrix entered via the log link with log(S) as an offset (log(μ)=Xβ+log(S)). For each feature, fit the Compound Poisson Linear Model by profile likelihood using the tweedie, cplm, and statmod R packages, initializing the index parameter at p = 1.5 with a Compound Poisson GLM; if excess zeros are expected, fit the zero-inflated variant (ZICP) via Newton-Raphson on the ZICP likelihood. Test H0: β_g = 0 per feature with the Wald test, which follows an asymptotic χ²-distribution with 1 degree of freedom under the null. Adjust p-values for FDR and call features differentially expressed when the q-value falls below α = 0.05. The Tweedie distribution with 1 < p < 2 (Compound Poisson-Gamma) is the natural choice because it reduces to a semi-continuous distribution with a point mass at zero, capturing both heavy tails and zero-inflation in a single zero-adjusted model rather than separately modeling zeroes and non-zeroes.

## Related tools

- **Tweedieverse** (Core R/Bioconductor package fitting Tweedie (CPLM/ZICP) GLMs for differential analysis of semi-continuous omics data) — https://github.com/himelmallick/Tweedieverse
- **tweedie (R package)** (Evaluates the Tweedie density function (via Fourier inversion) for profile likelihood computation)
- **cplm (R package)** (Fits the initial Compound Poisson GLM at p = 1.5 used to initialize coefficients)
- **statmod (R package)** (Supports the computation of Tweedie likelihood components)
- **R** (Runtime environment; glm() and optim() functions of the stats base R package used in the fitting procedure)

## Examples

```
library(Tweedieverse); output = Tweedieverse(feature_table, metadata, test = "tweedie", cutoff = 0.05)
```

## Evaluation signals

- de_results.csv exists and contains coefficient, p-value, q-value, fold-change, and estimated index-parameter (p) columns for every feature
- q-values are monotone FDR-adjusted p-values, and significant calls are thresholded at α = 0.05
- Estimated per-feature index parameters fall in the range 1 < p < 2, confirming the semi-continuous Compound Poisson-Gamma regime
- Wald test p-values are computed per feature against H0: β_g = 0 with the χ²(1) asymptotic null distribution
- Benchmarking against ≥10 DE methods (MAST, edgeR, DESeq2, metagenomeSeq, monocle, etc.) shows competitive sensitivity and controlled FDR on simulated datasets

## Limitations

- Tweedieverse is not architected for the situation when zero-inflation is a property specific only to a subset of the genes
- No borrowed-information/shrinkage estimates for improved DE gene ranking; benefits of regularization/pooling untested
- Extension to multi-subject experimental designs, individual-level DE, and differential subpopulation analysis is not yet done
- Application to other modalities (metagenomics, metatranscriptomics, spatial transcriptomics) remains a conjecture, not fully validated

## Evidence

- [methods] Profile likelihood estimation and initialization at p=1.5: [evidence span withheld — anchor status: not_found]
