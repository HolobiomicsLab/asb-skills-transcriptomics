---
name: log-cosh-edge-preserving-regularization
description: Use when a gene-by-cell relative expression matrix (counts minus a normal-cell
  baseline) must be denoised before copy number segmentation while preserving piecewise-constant
  discontinuities (breakpoints) along genomic coordinates.
when_to_use_negative:
- The sample has no aneuploidy signal (tumors with minimal genomic alterations such
  as leukemias, pediatric cancers, or Ependymomas), where the copy-number-based smoothing-segmentation
  framework is not suited
- No relative expression matrix exists yet — the baseline subtraction step must precede
  smoothing
- The goal is segmentation itself rather than pre-segmentation denoising; use the
  Mumford-Shah greedy segmentation skill instead
edam_operation: http://edamontology.org/operation_3675
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3518
tools:
- name: SCEVAN
  role: Package containing classifyTumorCells with the SMOOTH option implementing
    the TV logcosh edge-preserving smoothing before segmentation
  repo: github.com/AntonioDeFalco/SCEVAN
- name: yaGST
  role: Mann-Whitney-Wilcoxon gene set test used upstream to identify confident normal
    cells whose median expression defines the baseline subtracted before smoothing
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-36790-9
    title: A variational algorithm to detect the clonal copy number substructure of
      tumors from scRNA-seq data
  source_content_hash: sha256:d80a8d0c01c5ddfe846aed43cc95ecde2dfcf1ebbe95427de9a3e35d703c2149
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SCEVAN
  - yaGST
  edam_operation: http://edamontology.org/operation_3675
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# log-cosh-edge-preserving-regularization

## Summary

Applies an edge-preserving total-variation (TV) smoothing with the phi(x)=logcosh(x) prior to the relative expression matrix, denoising the per-cell expression signal while retaining copy number breakpoints before segmentation, as implemented inside SCEVAN::classifyTumorCells.

## When to use

Use when a gene-by-cell relative expression matrix (counts minus a normal-cell baseline) must be denoised before copy number segmentation while preserving piecewise-constant discontinuities (breakpoints) along genomic coordinates.

## When NOT to use

- The sample has no aneuploidy signal (tumors with minimal genomic alterations such as leukemias, pediatric cancers, or Ependymomas), where the copy-number-based smoothing-segmentation framework is not suited
- No relative expression matrix exists yet — the baseline subtraction step must precede smoothing
- The goal is segmentation itself rather than pre-segmentation denoising; use the Mumford-Shah greedy segmentation skill instead

## Inputs

- Gene-by-cell relative expression matrix (count matrix minus copy number baseline of confident normal cells), genes sorted by genomic coordinates
- beta_vega regularization parameter (default 0.5)

## Outputs

- Smoothed relative expression matrix that is piecewise constant along the genomic axis with retained copy number breakpoints
- Smoothed matrix fed to greedy Mumford-Shah multichannel segmentation

## How to apply

After deriving the relative expression matrix by subtracting the copy number baseline (median expression of confident normal cells identified with getConfidentNormalCells at P < 1e-10 and NES > 1.0, or a synthetic gaussian baseline if none exist), apply the edge-preserving filter that minimizes the total variation functional with the phi(x)=logcosh(x) prior, chosen because it yields a well-posed minimization problem. Solve via a stable finite-difference iterative scheme on the gene-by-cell matrix until convergence, treating the regularization beta (beta_vega, default 0.5; higher beta gives coarser segmentation) as the only smoothing parameter. Invoke inside SCEVAN::classifyTumorCells with SMOOTH=TRUE before greedy Mumford-Shah segmentation. Validate that the output is piecewise constant along the genomic axis with breakpoints between segments retained while noise is regularized.

## Related tools

- **SCEVAN** (Package containing classifyTumorCells with the SMOOTH option implementing the TV logcosh edge-preserving smoothing before segmentation) — github.com/AntonioDeFalco/SCEVAN
- **yaGST** (Mann-Whitney-Wilcoxon gene set test used upstream to identify confident normal cells whose median expression defines the baseline subtracted before smoothing)

## Examples

```
results <- SCEVAN::pipelineCNA(count_mtx, sample = "MGH106", par_cores = 20, SUBCLONES = TRUE, plotTree = TRUE)
```

## Evaluation signals

- Smoothed matrix is piecewise constant along the genomic axis
- Discontinuities (copy number breakpoints) between segments are retained while noise is suppressed
- The finite-difference iterative updates converge before segmentation proceeds
- Only beta_vega controls smoothing strength; varying beta changes coarseness (higher beta = coarser segmentation)
- Downstream Mumford-Shah segmentation produces consistent segments shared across cells of the same clone

## Limitations

- beta is the only parameter and its exact value is not fixed in the article; it must be tuned (higher beta for more coarse-grained segmentation)
- Aneuploidy-based regularization/segmentation is unsuitable for tumors with minimal genomic alterations (leukemia, pediatric cancers, Ependymomas)
- Smoothing quality depends on the upstream baseline estimate; wrong identification of normal cells propagates errors into the smoothed matrix

## Evidence

- [discussion] logcosh prior yields well-posed problem: [evidence span withheld — anchor status: not_found]
- [discussion] baseline subtraction to obtain relative matrix: "the copy number baseline, estimated from the median expression of confident normal cells, is removed from the count matrix, thus obtaining the relative matrix"
- [abstract] smoothed profile as evidence of CN profile: "Thus, the smoothed expression profile of every individual cell constitutes part of the evidence of the copy number profile in each subclone"
- [discussion] beta as only segmentation parameter: "where β is a positive constant, representing the only parameter of the segmentation algorithm."
- [methods] classifyTumorCells signature with SMOOTH and beta_vega: "- `classifyTumorCells(count_mtx, annot_mtx, sample, distance, par_cores, ground_truth, norm_cell_names, SEGMENTATION_CLASS, SMOOTH, beta_vega, FIXED_NORMAL_CELLS, output_dir)`"
- [readme] beta_vega default and meaning in README: [evidence span withheld — anchor status: external_doc]
