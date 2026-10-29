---
name: roi-boundary-zero-padding
description: Use when per-ROI histogram-equalized imputation patches (from SERM's
  sliding-window stage) must be merged into a single expression matrix with globally
  consistent expression values.
when_to_use_negative:
- The expression matrix is too small to divide into N >= 4 ROIs, so there are no patch
  seams to reconcile via resampling
- The data are not a sparse scRNA-seq count/expression matrix requiring SERM-style
  patch-wise imputation (e.g. already a dense feature table)
- Only a single-ROI imputation was performed, in which case no cross-patch consistency
  step is needed
tools:
- name: self-consistent expression recovery machine (SERM)
  role: The imputation framework whose final step is the bilinear resampling of imputed
    ROIs with zero-padded boundaries; the resampling merges the per-ROI patches produced
    by SERM's sliding-window histogram equalization.
  repo: https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-34595-w
    title: Leveraging data-driven self-consistency for high-fidelity gene expression
      recovery
  source_content_hash: sha256:79047999720887b158d56203cefdcad3e143dd010efc08d07a5e46d1df94d831
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
  tools:
  - self-consistent expression recovery machine (SERM)
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# roi-boundary-zero-padding

## Summary

Final consistency step of the SERM imputation pipeline: all per-ROI imputed patches are merged via bilinear resampling, where each interior element is the sample average of its four diagonal neighbors and matrix-edge elements use zero padding (s_hat_i0 = s_hat_0j = s_hat_i(q+1) = s_hat_(n+1)j = 0).

## When to use

Use when per-ROI histogram-equalized imputation patches (from SERM's sliding-window stage) must be merged into a single expression matrix with globally consistent expression values.

## When NOT to use

- The expression matrix is too small to divide into N >= 4 ROIs, so there are no patch seams to reconcile via resampling
- The data are not a sparse scRNA-seq count/expression matrix requiring SERM-style patch-wise imputation (e.g. already a dense feature table)
- Only a single-ROI imputation was performed, in which case no cross-patch consistency step is needed

## Inputs

- Per-ROI imputed expression matrix from the SERM ROI histogram-equalization stage (q cells x m genes)
- Original observed (dropout-affected) scRNA-seq data matrix for dimension checking
- ROI size (e.g. rows/4 or rows/8 by genes/4 or genes/8) and percent overlap between successive ROIs (0.01-0.99; 0.25-0.75 recommended)

## Outputs

- Final imputed gene expression matrix S_bar with exactly q rows and m columns matching the observed matrix
- imputed_matrix.csv file containing the resampled matrix

## How to apply

Load the per-ROI imputed expression matrix produced by the ROI histogram-equalization stage and confirm dimensions q x m against the observed data matrix. For every interior element, compute the resampled value as the four-diagonal-neighbor average: s_bar_ij = (1/4)(s_hat_(i-1)(j-1) + s_hat_(i-1)(j+1) + s_hat_(i+1)(j+1) + s_hat_(i+1)(j-1)). For boundary elements (first/last rows and columns), apply zero padding consistent with bilinear interpolation of continuous functions of two variables: s_hat_i0 = s_hat_0j = s_hat_i(q+1) = s_hat_(n+1)j = 0 for all i and j. Assemble the resampled matrix S_bar as the final imputed gene expression matrix and save it as imputed_matrix.csv. This step exists because SERM divides the matrix into N >= 4 ROIs and slides the ROI along the x- and y-axes, so patch-wise imputations must be reconciled into one globally consistent matrix.

## Related tools

- **self-consistent expression recovery machine (SERM)** (The imputation framework whose final step is the bilinear resampling of imputed ROIs with zero-padded boundaries; the resampling merges the per-ROI patches produced by SERM's sliding-window histogram equalization.) — https://github.com/xinglab-ai/self-consistent-expression-recovery-machine

## Examples

```
import serm as srm; out_SERM = srm.serm(dataObs, ROIsize, percOL, randomize=False)
```

## Evaluation signals

- Output matrix has exactly q rows and m columns matching the observed matrix dimensions
- Each interior resampled value equals the four-diagonal-neighbor average (1/4)(s_hat_(i-1)(j-1) + s_hat_(i-1)(j+1) + s_hat_(i+1)(j+1) + s_hat_(i+1)(j-1))
- Boundary values (first/last row and column) follow the zero-padding rule s_hat_i0 = s_hat_0j = s_hat_i(q+1) = s_hat_(n+1)j = 0
- Resulting matrix is assembled and saved as imputed_matrix.csv without NaNs or dimension mismatches
- Downstream checks: imputed data recovers expression distributions similar to the reference (e.g. similar number of zeros to reference data, Pearson coefficient > 0.9 at 5% sampling efficiency)

## Limitations

- Boundary rows and columns carry no resampling information (zero padding), so edge imputations rely entirely on the zero-padding convention rather than neighbor averaging
- The upstream ROI-based imputation assumes ROI coverage is sufficiently large that the captured distribution is representative of the entire matrix; small datasets may yield inaccurate imputed values
- Batch effects are not considered explicitly in the SERM formulation, and SERM learns its model from a small fraction of the data

## Evidence

- [intro] bilinear resampling merges patches: "In the last step, all the ROIs are interpolated using bilinear resampling to achieve the global consistency of the expression values"
- [methods] resampling for patch consistency: "We now incorporate the histograms of different ROIs into a consistent framework by applying a bilinear resampling."
- [methods] matrix division into ROIs (why seams exist): "We divide the expression matrix S is divided into N ≥ 4 ROIs, which we denote by Si, i = 1, 2, ⋯ , N."
- [intro] SERM workflow: sliding ROI imputation upstream of resampling: "The ROI then slides along the x and y direction throughout the expression matrix, and histogram equalization is performed on each ROI."
- [readme] README: input matrix layout and ROI/overlap parameters: "The gene expression data should be in matrix format (rows are cells and collumns are genes)."
