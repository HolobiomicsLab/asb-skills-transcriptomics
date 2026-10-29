---
name: tweedie-model-gene-level-fitting
description: Use when performing differential expression on scRNA-seq UMI or non-UMI
  count matrices (or other omics counts) between two conditions, especially when data
  are sparse, zero-inflated, or heavy-tailed and standard NB-based methods underperform.
when_to_use_negative:
- Zero-inflation is a property specific to only a subset of genes — the paper states
  Tweedieverse 'is not architected for the situation when zero-inflation is a property
  specific only to a subset of the genes'
- Multi-subject experimental designs, individual-level differential expression, or
  differential subpopulation analysis — extension and validation of Tweedieverse to
  these settings had not been done
- Continuous measurements where the analysis goal requires shrinkage/borrowed-information
  gene ranking — the article notes no shrinkage estimates for improved DE gene ranking
  are implemented
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Tweedieverse
  role: R/Bioconductor package that fits per-gene CPLM/ZICP models and performs Wald-test
    differential expression
  repo: https://github.com/himelmallick/Tweedieverse
- name: tweedie (R package)
  role: Evaluates the Tweedie density function via Fourier inversion during likelihood
    computation
- name: cplm (R package)
  role: Used with stats base R glm()/optim() to carry out the Compound Poisson model
    computations
- name: statmod (R package)
  role: Supports the numerical computation of the Tweedie likelihood
- name: SCRAN
  role: Estimates cell-specific library sizes incorporated as offsets in the per-gene
    Tweedie models
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/sim.9430
    title: Differential expression of single‐cell RNA‐seq data using Tweedie models
  source_content_hash: sha256:44be5dda233dc07f481f7d46687a40b08137f989257e2b01598631a97cd259d5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/himelmallick/Tweedieverse
  tools:
  - Bioconductor
  - R
  - SCRAN
  - Tweedieverse
  - cplm
  - cplm (R package)
  - statmod
  - statmod (R package)
  - tweedie
  - tweedie (R package)
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# tweedie-model-gene-level-fitting

## Summary

Fit per-gene generalized linear models with a Tweedie-distributed response (CPLM for UMI counts, ZICP for zero-inflated read counts) using the Tweedieverse R/Bioconductor package, then call differentially expressed genes via Wald tests with FDR correction. The three-parameter Tweedie family captures heavy tails and zero-inflation of scRNA-seq data in a single zero-adjusted model, bypassing ad hoc normalization or transformation.

## When to use

Use when performing differential expression on scRNA-seq UMI or non-UMI count matrices (or other omics counts) between two conditions, especially when data are sparse, zero-inflated, or heavy-tailed and standard NB-based methods underperform.

## When NOT to use

- Zero-inflation is a property specific to only a subset of genes — the paper states Tweedieverse 'is not architected for the situation when zero-inflation is a property specific only to a subset of the genes'
- Multi-subject experimental designs, individual-level differential expression, or differential subpopulation analysis — extension and validation of Tweedieverse to these settings had not been done
- Continuous measurements where the analysis goal requires shrinkage/borrowed-information gene ranking — the article notes no shrinkage estimates for improved DE gene ranking are implemented

## Inputs

- Gene-by-cell raw count matrix (UMI or non-UMI scRNA-seq counts), genes filtered to those expressed in at least 20% of cells
- Sample/cell metadata table with binary condition labels and covariates
- Cell-specific library sizes (SCRAN-estimated) used as model offsets

## Outputs

- Per-gene Wald test p-values from an asymptotic χ2 distribution with 1 degree of freedom
- FDR-adjusted q-values and DE calls at α = 0.05
- Per-gene estimates of mean (μ), dispersion (φ), and index/power parameter (p)

## How to apply

Model the per-gene count vector Y as either a Tweedie (CPLM) or zero-inflated Tweedie (ZICP) distribution with mean μ, dispersion φ, and index parameter p constrained to (1, 2) with dispersion > 0; in this range the Tweedie distribution reduces to a semi-continuous Compound Poisson-Gamma with a point mass at zero. Include cell-specific library size S as an offset via log(μ)=Xβ+log(S), estimating S with the SCRAN R/Bioconductor package rather than applying a separate normalization or transformation. Estimate the index and dispersion parameters by profiling out β and maximizing the profile likelihood (evaluating the Tweedie density via Fourier inversion using the tweedie, cplm, and statmod R packages); initialize with a Compound Poisson GLM at p = 1.5 and fit ZICP parameters via Newton-Raphson using glm() and optim() from stats base R. Choose CPLM for UMI counts and ZICP for non-UMI read counts; compute per-gene Wald test p-values (asymptotic χ2 with 1 degree of freedom) and declare genes differentially expressed when the FDR-adjusted q-value falls below α = 0.05.

## Related tools

- **Tweedieverse** (R/Bioconductor package that fits per-gene CPLM/ZICP models and performs Wald-test differential expression) — https://github.com/himelmallick/Tweedieverse
- **tweedie (R package)** (Evaluates the Tweedie density function via Fourier inversion during likelihood computation)
- **cplm (R package)** (Used with stats base R glm()/optim() to carry out the Compound Poisson model computations)
- **statmod (R package)** (Supports the numerical computation of the Tweedie likelihood)
- **SCRAN** (Estimates cell-specific library sizes incorporated as offsets in the per-gene Tweedie models)

## Evaluation signals

- Fitted index parameter p falls within (1, 2) and dispersion φ > 0 for every gene model; convergence achieved via profile likelihood optimization
- Wald test statistic under the null follows an asymptotic χ2 distribution with 1 degree of freedom; q-values are FDR-adjusted before calling DE at α = 0.05
- Per-gene models show high F1 and AUC scores with controlled FDR in the benchmark, with ZICP slightly better than CPLM on zero-inflated non-UMI counts
- Reproduced comparison against >10 baselines (DESeq2, edgeR, MAST, DEsingle, etc.) via the BenchmarkSingleCell code confirms Tweedieverse outperforms state-of-the-art DE methods across plate- and droplet-based platforms

## Limitations

- Tweedieverse is not architected for situations where zero-inflation is specific to only a subset of genes
- No borrowed-information/shrinkage estimates are available to improve DE gene ranking; benefits of regularization/pooling remain untested
- Lack of interoperability between platform-specific methods remains a concern; extension to multi-subject designs, other modalities (metagenomics, spatial transcriptomics) is a conjecture, not validated
- Fitting requires numerical evaluation of the Tweedie density via Fourier inversion, which can be computationally demanding

## Evidence

- [methods] Model gene expression with Tweedie (CPLM) or zero-inflated Tweedie (ZICP) distribution: [evidence span withheld — anchor status: not_found]
- [readme] Single zero-adjusted model rationale: [evidence span withheld — anchor status: not_found]
