---
name: cross-platform-scrnaseq-parameter-comparison
description: Use when multiple publicly available scRNA-seq count datasets from different
  protocols (e.g., UMI-based 10x/droplet vs. plate-based read counts) are available
  and the goal is to test whether distributional assumptions such as the Tweedie index
  p differ by platform.
when_to_use_negative:
- The analysis goal is differential expression testing with a single fixed p rather
  than characterizing platform- and gene-specific variation of the index parameter
- Zero-inflation is expected to be a property specific only to a subset of genes —
  Tweedieverse 'is not architected for the situation when zero-inflation is a property
  specific only to a subset of the genes'
- Data are already normalized/transformed expression values without raw counts and
  library-size offsets; the Tweedie models rely on raw counts with SCRAN-derived offsets
  in the log link
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Tweedieverse
  role: R/Bioconductor package used to fit per-gene CPLM (UMI counts) and ZICP (read
    counts) models and estimate the Tweedie index and dispersion parameters
  repo: https://github.com/himelmallick/Tweedieverse
- name: tweedie (R package)
  role: Evaluates the Tweedie density function via Fourier inversion during likelihood
    computation
- name: cplm (R package)
  role: Compound Poisson GLM machinery used for initialization and likelihood computation
    in the CPLM fits
- name: SCRAN
  role: Estimates cell-specific library sizes incorporated as offsets in the per-gene
    Tweedie models
- name: statmod (R package)
  role: Supporting package for the Tweedie density/parameter estimation computation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/sim.9430
    title: Differential expression of single‐cell RNA‐seq data using Tweedie models
  source_content_hash: sha256:b31f4991f7b1e1172ac41abe77bcbdf538693133da1ddd11f57e6398fbe35a2e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/himelmallick/Tweedieverse
  tools:
  - Bioconductor
  - R
  - SCRAN
  - Tweedieverse
  - cplm (R package)
  - statmod (R package)
  - tweedie (R package)
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# cross-platform-scrnaseq-parameter-comparison

## Summary

Estimate gene-specific Tweedie index parameter p per scRNA-seq dataset and compare its distribution across experimental platforms (UMI vs. read-count protocols), demonstrating that a fixed p is inadequate and motivating a flexible Tweedie modeling framework (Tweedieverse).

## When to use

Use when multiple publicly available scRNA-seq count datasets from different protocols (e.g., UMI-based 10x/droplet vs. plate-based read counts) are available and the goal is to test whether distributional assumptions such as the Tweedie index p differ by platform.

## When NOT to use

- The analysis goal is differential expression testing with a single fixed p rather than characterizing platform- and gene-specific variation of the index parameter
- Zero-inflation is expected to be a property specific only to a subset of genes — Tweedieverse 'is not architected for the situation when zero-inflation is a property specific only to a subset of the genes'
- Data are already normalized/transformed expression values without raw counts and library-size offsets; the Tweedie models rely on raw counts with SCRAN-derived offsets in the log link

## Inputs

- Public scRNA-seq raw count matrices (UMI counts and non-UMI read counts) from GEO accessions such as GSE67835 and GSE107585 and the 10x Genomics PBMC dataset (13,713 genes, 2,638 cells)
- Per-cell metadata (experimental protocol, dataset/protocol labels, batch/control annotations)

## Outputs

- Per-gene, per-dataset estimates of the Tweedie index parameter p (all within (1, 2)) and dispersion phi
- Cross-protocol comparison of p distributions (analogous to Fig. 1 of the paper), supporting platform-specific Tweedie parameterization

## How to apply

Download the 16 public scRNA-seq datasets (e.g., GEO GSE67835 brain data, GSE107585 mouse kidney, 10x Genomics PBMC with 13,713 genes × 2,638 cells), distinguishing UMI count data from read count data. Estimate cell-specific library sizes with SCRAN and include them as offsets under the log link log(mu) = X*beta + log(S). For each gene, fit CPLM for UMI counts or ZICP for read counts with Tweedieverse, initializing coefficients with a Compound Poisson GLM at p = 1.5, evaluating the Tweedie density by Fourier inversion (tweedie package), and optimizing (phi, p) by profile likelihood (CPLM) or Newton-Raphson (ZICP) subject to phi > 0 and p in (1, 2). Compare per-gene p estimate distributions across protocols; the Tweedie density has a point mass at zero only when 1 < p < 2 (Compound Poisson-Gamma), which is why that constrained range is enforced. Correctness is judged by all estimates falling in (1, 2) and by reproducing significant cross-platform differences (Fig. 1 of the paper).

## Related tools

- **Tweedieverse** (R/Bioconductor package used to fit per-gene CPLM (UMI counts) and ZICP (read counts) models and estimate the Tweedie index and dispersion parameters) — https://github.com/himelmallick/Tweedieverse
- **tweedie (R package)** (Evaluates the Tweedie density function via Fourier inversion during likelihood computation)
- **cplm (R package)** (Compound Poisson GLM machinery used for initialization and likelihood computation in the CPLM fits)
- **SCRAN** (Estimates cell-specific library sizes incorporated as offsets in the per-gene Tweedie models)
- **statmod (R package)** (Supporting package for the Tweedie density/parameter estimation computation)

## Examples

```
library(Tweedieverse); out <- Tweedieverse(feature_table = counts, metadata = meta)
```

## Evaluation signals

- All per-gene p estimates lie strictly within the constrained range (1, 2)
- All dispersion estimates satisfy phi > 0
- Distributions of gene-specific p estimates differ significantly across the 16 datasets/protocols, reproducing the cross-protocol separation reported in Fig. 1
- UMI datasets are fit with CPLM and read-count datasets with ZICP, and optimization converges (profile likelihood for CPLM, Newton-Raphson for ZICP)
- Library-size offsets are SCRAN-derived and enter the model as log(S) offsets under the log link

## Limitations

- Tweedieverse is not architected for the situation when zero-inflation is a property specific only to a subset of the genes
- No borrowed-information/shrinkage estimates are available for improved DE gene ranking; benefits of regularization/pooling are untested
- Extension to multi-subject designs, individual-level DE, differential subpopulation analysis, and other modalities (metagenomics, spatial transcriptomics) remains unvalidated conjecture

## Evidence

- [intro] significant cross-protocol p differences: "we found significant differences in gene-specific estimates of p when we considered 16 publicly available scRNA-seq datasets across experimental protocols"
- [results] SCRAN offsets in Tweedie models: "we first estimate cell-specific library sizes using the SCRAN45 R/Bioconductor package and incorporate them as an offset in the per-gene Tweedie models"
- [methods] initialization at p = 1.5: "the index parameter is first set to 1.5 and an initial Compound Poisson GLM is fit with p = 1.5"
- [methods] ZICP Newton-Raphson estimation: "For the zero-inflated Tweedie model, we employ a Newton-Raphson method"
- [readme] technology-agnostic scope: "applied to both count and continuous measurements arising from diverse high-throughput technologies"
