---
name: scrnaseq-library-size-offset-normalization
description: Use when analyzing scRNA-seq count matrices (UMI or read counts) across
  cells with varying sequencing depth, and a Tweedie/GLM-style model requires a library-size
  offset log(S) instead of pre-normalizing counts.
when_to_use_negative:
- Input is already normalized or log-transformed expression data — adding a log(S)
  offset would double-correct for sequencing depth; the article's approach explicitly
  bypasses ad hoc normalization/transformation prior to DE testing.
- Data lack per-cell library size information (e.g., pre-aggregated pseudobulk or
  metacell matrices) and the log link model log(μ)=Xβ+log(S) cannot be specified.
- Zero-inflation is a property specific only to a subset of genes, a situation the
  authors state Tweedieverse is not architected for.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SCRAN
  role: Estimates cell-specific library sizes from the raw count matrix, which are
    then incorporated as offsets in the per-gene Tweedie models
- name: Tweedieverse
  role: Fits per-gene Tweedie (CPLM) and zero-inflated Tweedie (ZICP) models with
    log(μ)=Xβ+log(S), carrying the library-size offset through to DE testing
  repo: https://github.com/himelmallick/Tweedieverse
- name: cplm (R package)
  role: Supports the Compound Poisson Linear Model fitting machinery used with the
    library-size offset in the Tweedie framework
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/sim.9430
    title: Differential expression of single‐cell RNA‐seq data using Tweedie models
  source_content_hash: sha256:74491e5d3aa49e57d728fe1be5839a6102af17bc67c7c8aa79870c4ef6d35bb4
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
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# scrnaseq-library-size-offset-normalization

## Summary

Normalizes single-cell RNA-seq expression data for sequencing depth by estimating cell-specific library sizes with the SCRAN R/Bioconductor package and incorporating them as an additive offset (log(S)) in per-gene Tweedie models (CPLM/ZICP), bypassing ad hoc normalization or transformation prior to differential expression testing.

## When to use

Use when analyzing scRNA-seq count matrices (UMI or read counts) across cells with varying sequencing depth, and a Tweedie/GLM-style model requires a library-size offset log(S) instead of pre-normalizing counts.

## When NOT to use

- Input is already normalized or log-transformed expression data — adding a log(S) offset would double-correct for sequencing depth; the article's approach explicitly bypasses ad hoc normalization/transformation prior to DE testing.
- Data lack per-cell library size information (e.g., pre-aggregated pseudobulk or metacell matrices) and the log link model log(μ)=Xβ+log(S) cannot be specified.
- Zero-inflation is a property specific only to a subset of genes, a situation the authors state Tweedieverse is not architected for.

## Inputs

- Raw scRNA-seq count matrix (UMI counts, e.g., 10x Genomics PBMC 13,713 genes × 2,638 cells, or non-UMI read counts)
- Cell-level metadata/design matrix X for the linear predictor
- SCRAN-estimated cell-specific library sizes or normalization factors (derived from the count matrix)

## Outputs

- Per-gene Tweedie model fits (CPLM or ZICP) with library size as an offset
- Estimated coefficients, dispersion (φ), and index parameter p within (1, 2)
- Wald test p-values and FDR-adjusted q-values for differential expression

## How to apply

Estimate cell-specific library sizes from the raw count matrix using the SCRAN R/Bioconductor package. In the per-gene Tweedie model, fit the mean via the log link log(μ)=Xβ+log(S), where S is the library size (or normalization factor) entered as a model offset — this borrows information across cells and avoids ad hoc normalization or transformation before DE testing. Use the offset consistently whether fitting the non-zero-inflated Compound Poisson Linear Model (CPLM) for UMI counts or the zero-inflated variant (ZICP) for read counts, with coefficients initialized from a Compound Poisson GLM at p = 1.5. The offset is applied before Wald testing and FDR control at α = 0.05, so ensure S is available and positive for every cell in the design.

## Related tools

- **SCRAN** (Estimates cell-specific library sizes from the raw count matrix, which are then incorporated as offsets in the per-gene Tweedie models)
- **Tweedieverse** (Fits per-gene Tweedie (CPLM) and zero-inflated Tweedie (ZICP) models with log(μ)=Xβ+log(S), carrying the library-size offset through to DE testing) — https://github.com/himelmallick/Tweedieverse
- **cplm (R package)** (Supports the Compound Poisson Linear Model fitting machinery used with the library-size offset in the Tweedie framework)

## Evaluation signals

- Every fitted model contains the offset term log(S) with positive, cell-specific library sizes (no missing or non-finite S values).
- Per-gene index parameter estimates p remain within the constrained range (1, 2) across all datasets after offset-based fitting.
- DE calls are based on FDR-adjusted p-values (q-values) thresholded at α = 0.05, consistent with the paper's decision rule.
- Results should show no residual sequencing-depth association (depth handled by the offset, not by pre-transformation of counts).
- Reproduction of the paper's pattern: significant differences in gene-specific p estimates across the 16 datasets when offsets are applied consistently.

## Limitations

- Version numbers for SCRAN are not stated in the paper's results section.
- The offset approach is embedded in the Tweedieverse framework; the authors note Tweedieverse is not architected for zero-inflation specific to only a subset of genes.
- Benefits of regularization or pooling strategies for the proposed offset-based models remain untested per the discussion.

## Evidence

- [readme] Offset replaces ad hoc normalization: [evidence span withheld — anchor status: not_found]
