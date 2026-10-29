---
name: breakpoint-preservation-qc-in-cnv-matrices
description: Use when a gene-by-cell relative expression matrix (after baseline subtraction
  from confident normal cells) must be denoised before copy number segmentation while
  retaining breakpoints needed for joint segmentation and clone detection.
when_to_use_negative:
- The tumor has minimal genomic alterations (e.g. leukemia, pediatric cancers, Ependymomas)
  where aneuploidy-based smoothing and segmentation assumptions are unsuitable
- A discretized or already-segmented CNA matrix is provided — smoothing a piecewise-constant
  input would erode genuine breakpoints
- Only normal/tumor cell classification is needed with no downstream segmentation,
  when the smoothing step's main output (breakpoint-preserving profiles) is not required
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SCEVAN
  role: Provides the SMOOTH option inside classifyTumorCells that performs the edge-preserving
    logcosh total variation smoothing of the relative expression matrix before segmentation;
    beta_vega controls the regularization strength
  repo: github.com/AntonioDeFalco/SCEVAN
- name: yaGST
  role: Mann-Whitney-Wilcoxon single-sample gene set test used in getConfidentNormalCells
    to identify the confident normal cells whose median expression defines the baseline
    subtracted prior to smoothing
  repo: github.com/miccec/yaGST
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-36790-9
    title: A variational algorithm to detect the clonal copy number substructure of
      tumors from scRNA-seq data
  source_content_hash: sha256:03ca5844b4d0a01ed32f1edf2f1ea28c358c208d743060876e08c32ade073fb2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SCEVAN
  - yaGST
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# breakpoint-preservation-qc-in-cnv-matrices

## Summary

Applies SCEVAN's edge-preserving nonlinear diffusion smoothing (total variation with phi(x)=logcosh(x) prior) to the gene-by-cell relative expression matrix so noise is regularized while copy number breakpoints are retained before Mumford-Shah segmentation. The smoothing quality is then verified by checking the output is piecewise constant along the genomic axis with breakpoints preserved.

## When to use

Use when a gene-by-cell relative expression matrix (after baseline subtraction from confident normal cells) must be denoised before copy number segmentation while retaining breakpoints needed for joint segmentation and clone detection.

## When NOT to use

- The tumor has minimal genomic alterations (e.g. leukemia, pediatric cancers, Ependymomas) where aneuploidy-based smoothing and segmentation assumptions are unsuitable
- A discretized or already-segmented CNA matrix is provided — smoothing a piecewise-constant input would erode genuine breakpoints
- Only normal/tumor cell classification is needed with no downstream segmentation, when the smoothing step's main output (breakpoint-preserving profiles) is not required

## Inputs

- Gene-by-cell raw count matrix (genes on rows, Gene Symbol or Ensembl ID allowed) e.g. MGH106 from GSE131928 glioblastoma scRNA-seq
- Relative expression matrix obtained after subtracting the median-expression copy number baseline of confident normal cells

## Outputs

- Smoothed relative expression matrix that is piecewise constant along the genomic axis with copy number breakpoints retained
- Smoothed per-cell expression profiles serving as evidence of the copy number profile in each subclone for subsequent Mumford-Shah segmentation

## How to apply

First derive the relative expression matrix by subtracting the copy number baseline, estimated as the median expression of confident normal cells identified with SCEVAN::getConfidentNormalCells (P < 1e-10, NES > 1.0, max 30 cells); if none are found, fall back to removing a synthetic gaussian baseline assuming a pure sample. On this matrix run the edge-preserving smoothing that minimizes the total variation functional with the phi(x)=logcosh(x) prior, chosen because it yields a well-posed minimization problem, solved with a stable finite-difference iterative scheme as used inside SCEVAN::classifyTumorCells with the SMOOTH option. Keep beta (beta_vega, default 0.5) as the only smoothing parameter; higher beta gives more coarse-grained segmentation. Iterate the finite-difference updates until convergence, then validate that the output matrix is piecewise constant along the genomic axis, retaining breakpoints between segments while regularizing noise, before handing off to the greedy Mumford-Shah multichannel segmentation.

## Related tools

- **SCEVAN** (Provides the SMOOTH option inside classifyTumorCells that performs the edge-preserving logcosh total variation smoothing of the relative expression matrix before segmentation; beta_vega controls the regularization strength) — github.com/AntonioDeFalco/SCEVAN
- **yaGST** (Mann-Whitney-Wilcoxon single-sample gene set test used in getConfidentNormalCells to identify the confident normal cells whose median expression defines the baseline subtracted prior to smoothing) — github.com/miccec/yaGST

## Examples

```
results <- SCEVAN::pipelineCNA(count_mtx, sample = "MGH106", par_cores = 20, SUBCLONES = TRUE, plotTree = TRUE)
```

## Evaluation signals

- The smoothed matrix is piecewise constant along the genomic (gene-sorted by coordinate) axis
- Breakpoints between segments are retained rather than smoothed away, so subsequent Mumford-Shah segmentation recovers segment boundaries
- The logcosh prior yields a well-posed minimization problem and the finite-difference iteration converges stably
- beta_vega is the only smoothing parameter (default 0.5); increasing beta produces a more coarse-grained segmentation as expected
- Downstream outputs (e.g. MGH106_CNAmtx.RData / .seg regions from pipelineCNA) show coherent per-subclone copy number profiles consistent with the smoothed profiles

## Limitations

- An aneuploidy-based approach (and this smoothing-before-segmentation design) may not be suited to tumors with minimal genomic alterations such as leukemia, pediatric cancers, and Ependymomas
- The value of the beta constant is not specified in the article text; it is only documented as beta_vega (default 0.5) in the README
- If no confident normal cells are found (P < 1e-10, NES > 1.0), a synthetic gaussian baseline must be removed instead, which changes the nature of the relative matrix being smoothed

## Evidence

- [discussion] logcosh prior yields well-posed minimization: [evidence span withheld — anchor status: not_found]
- [readme] beta_vega parameter semantics: [evidence span withheld — anchor status: not_found]
- [methods] classifyTumorCells signature with SMOOTH and beta_vega: "classifyTumorCells(count_mtx, annot_mtx, sample, distance, par_cores, ground_truth, norm_cell_names, SEGMENTATION_CLASS, SMOOTH, beta_vega, FIXED_NORMAL_CELLS, output_dir)"
- [discussion] baseline subtraction of confident normal cells: "the copy number baseline, estimated from the median expression of confident normal cells, is removed from the count matrix, thus obtaining the relative matrix"
- [abstract] smoothed profiles as CN evidence: "Thus, the smoothed expression profile of every individual cell constitutes part of the evidence of the copy number profile in each subclone"
