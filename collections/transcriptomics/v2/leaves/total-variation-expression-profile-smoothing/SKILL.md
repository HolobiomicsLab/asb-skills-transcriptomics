---
name: total-variation-expression-profile-smoothing
description: Use when a gene-by-cell relative expression matrix (baseline-subtracted
  scRNA-seq counts ordered by genomic position) must be denoised while preserving
  piecewise-constant breakpoints prior to copy number segmentation.
when_to_use_negative:
- The matrix has not yet been baseline-corrected into a relative expression matrix
  (no normal-cell median subtraction or gaussian baseline removal)
- The goal is fine-grained denoising that preserves per-gene continuous variation
  rather than piecewise-constant segments (higher beta values coarsen the segmentation)
- The tumor type has minimal genomic alterations (e.g. leukemia, pediatric cancers,
  ependymomas) where aneuploidy-based regularization is unsuitable
edam_operation: http://edamontology.org/operation_3675
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_2229
tools:
- name: SCEVAN
  role: Implements the TV smoothing with logcosh prior via classifyTumorCells (SMOOTH
    option) and getBreaksVegaMC with beta_vega, prior to Mumford–Shah joint segmentation
  repo: github.com/AntonioDeFalco/SCEVAN
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-36790-9
    title: A variational algorithm to detect the clonal copy number substructure of
      tumors from scRNA-seq data
  source_content_hash: sha256:ce1bede8fb4db3d3d2ef5c1c805b72968e20ffb761421cdfec24626238a810dd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SCEVAN
  edam_operation: http://edamontology.org/operation_3675
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# total-variation-expression-profile-smoothing

## Summary

Edge-preserving nonlinear diffusion smoothing of the relative expression matrix (genes x cells, sorted by genomic coordinate) inside SCEVAN, minimizing a total variation functional with a phi(x)=logcosh(x) prior so noise is regularized while copy number breakpoints are retained before joint segmentation.

## When to use

Use when a gene-by-cell relative expression matrix (baseline-subtracted scRNA-seq counts ordered by genomic position) must be denoised while preserving piecewise-constant breakpoints prior to copy number segmentation.

## When NOT to use

- The matrix has not yet been baseline-corrected into a relative expression matrix (no normal-cell median subtraction or gaussian baseline removal)
- The goal is fine-grained denoising that preserves per-gene continuous variation rather than piecewise-constant segments (higher beta values coarsen the segmentation)
- The tumor type has minimal genomic alterations (e.g. leukemia, pediatric cancers, ependymomas) where aneuploidy-based regularization is unsuitable

## Inputs

- Gene-by-cell relative expression matrix (raw scRNA-seq count matrix minus copy number baseline, e.g. MGH106 from GSE131928)
- Genomic annotation of genes (Ensembl-based locations, sorted by genomic coordinates)
- beta_vega regularization parameter (default 0.5)

## Outputs

- Smoothed relative expression matrix, piecewise constant along the genomic axis with retained breakpoints between segments
- Smoothed matrix used as input to greedy Mumford–Shah joint segmentation

## How to apply

Derive the relative expression matrix by subtracting the copy number baseline estimated as the median expression of confident normal cells (identified via SCEVAN::getConfidentNormalCells with P < 1e-10 and NES > 1.0), or, if no confident normal cells exist, remove a synthetic gaussian baseline assuming a pure sample. Then apply the smoothing step used inside SCEVAN::classifyTumorCells with the SMOOTH option: minimize the total variation functional with the log-cosh prior phi(x)=logcosh(x), solved by a stable finite-difference iterative scheme over the gene-by-cell matrix until convergence. Keep beta (beta_vega, default 0.5) as the only smoothing/segmentation parameter — higher beta yields a more coarse-grained segmentation. The log-cosh prior is chosen because it produces a well-posed minimization problem and yields a piecewise-constant result along the genomic axis.

## Related tools

- **SCEVAN** (Implements the TV smoothing with logcosh prior via classifyTumorCells (SMOOTH option) and getBreaksVegaMC with beta_vega, prior to Mumford–Shah joint segmentation) — github.com/AntonioDeFalco/SCEVAN

## Examples

```
results <- SCEVAN::pipelineCNA(count_mtx, sample = "MGH106", par_cores = 20, SUBCLONES = TRUE, beta_vega = 0.5)
```

## Evaluation signals

- The smoothed matrix is piecewise constant along the genomic axis
- Breakpoints between segments are retained (discontinuities survive smoothing), as required by the subsequent Mumford–Shah segmentation
- The log-cosh minimization converges under the stable finite-difference iterative scheme (well-posed minimization problem)
- Only beta_vega is tuned as the smoothing parameter (default 0.5); higher beta yields coarser-grained segmentation
- Downstream segmentation on the smoothed matrix does not produce artificial segments beyond those attributable to noise or cell cycle (cell cycle genes already filtered via REACTOME)

## Limitations

- beta_vega is the only smoothing parameter and its value trades off noise removal against breakpoint granularity; higher beta gives more coarse-grained segmentation
- Smoothing presumes the relative expression matrix was correctly baseline-corrected; if confident normal cells were misidentified (P < 1e-10, NES > 1.0 thresholds), the smoothed signal inherits the bias
- The overall aneuploidy-based approach may not suit tumors with minimal genomic alterations, limiting the utility of this smoothing step there

## Evidence

- [discussion] TV smoothing with logcosh prior before segmentation: [evidence span withheld — anchor status: not_found]
- [discussion] Baseline subtraction to obtain the relative expression matrix: "the copy number baseline, estimated from the median expression of confident normal cells, is removed from the count matrix, thus obtaining the relative matrix"
- [discussion] No confident normal cells fallback: "If no confident normal cells are found, we assume that the sample is pure and contains only malignant cells."
- [methods] beta_vega parameter in classifyTumorCells: "`classifyTumorCells(count_mtx, annot_mtx, sample, distance, par_cores, ground_truth, norm_cell_names, SEGMENTATION_CLASS, SMOOTH, beta_vega, FIXED_NORMAL_CELLS, output_dir)`"
- [readme] beta_vega default in README: [evidence span withheld — anchor status: not_found]
