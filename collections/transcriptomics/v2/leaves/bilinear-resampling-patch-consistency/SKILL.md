---
name: bilinear-resampling-patch-consistency
description: Use when a matrix has been imputed patch-wise (e.g., per-ROI histogram
  equalization in SERM) and the per-window outputs must be merged into a single globally
  consistent expression matrix before downstream analyses.
when_to_use_negative:
- The expression matrix has not been divided into at least 4 ROIs and imputed patch-wise,
  so there are no overlapping ROI histograms to reconcile
- The observed data are a small genomic dataset where SERM's estimated distribution
  may diverge from the actual data distribution, making the whole SERM imputation
  (and hence this final resampling) unreliable
- The input is already a single fully imputed matrix with consistent expression values
  across patches
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3170
tools:
- name: self-consistent expression recovery machine (SERM)
  role: Pipeline whose final 'Resampling For Patch Consistency' stage applies bilinear
    resampling to merge the per-ROI histograms into a consistent framework; installable
    as the Python package 'serm'
  repo: https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-34595-w
    title: Leveraging data-driven self-consistency for high-fidelity gene expression
      recovery
  source_content_hash: sha256:f6d200c880aab61f3fffbdc30cc7502ceda1300e7c532c6f335149a772e5bfab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
  tools:
  - self-consistent expression recovery machine (SERM)
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# bilinear-resampling-patch-consistency

## Summary

Final step of the SERM pipeline: after sliding-window histogram-equalization imputation of per-ROI patches, bilinear resampling merges all ROIs into one expression matrix whose values are globally consistent, computed as the average of four diagonal neighbors per element.

## When to use

Use when a matrix has been imputed patch-wise (e.g., per-ROI histogram equalization in SERM) and the per-window outputs must be merged into a single globally consistent expression matrix before downstream analyses.

## When NOT to use

- The expression matrix has not been divided into at least 4 ROIs and imputed patch-wise, so there are no overlapping ROI histograms to reconcile
- The observed data are a small genomic dataset where SERM's estimated distribution may diverge from the actual data distribution, making the whole SERM imputation (and hence this final resampling) unreliable
- The input is already a single fully imputed matrix with consistent expression values across patches

## Inputs

- Per-ROI imputed expression matrix (q x m, output of the SERM sliding-window histogram-equalization stage)
- Original observed data matrix (q x m) for dimension check

## Outputs

- Globally consistent imputed gene expression matrix S_bar (q x m), e.g., saved as imputed_matrix.csv

## How to apply

Load the per-ROI imputed matrix produced by the ROI histogram-equalization stage and confirm its dimensions are q x m, matching the original observed data. For every element, compute the resampled value as the sample average of the four diagonal neighbors: s_bar_ij = (1/4)(s_hat_(i-1)(j-1) + s_hat_(i-1)(j+1) + s_hat_(i+1)(j+1) + s_hat_(i+1)(j-1)), which treats the expression values as samples of a continuous function of two variables. Handle boundary elements (s_hat_i1, s_hat_iq, s_hat_1j, s_hat_nj) with zero padding: s_hat_i0 = s_hat_0j = s_hat_i(q+1) = s_hat_(n+1)j = 0 for all i and j. Assemble the resampled matrix S_bar as the final imputed gene expression matrix and save it (e.g., imputed_matrix.csv). In SERM this step follows ROI selection (N >= 4 ROIs), adaptive histogram equalization, and ROI sliding throughout the whole expression matrix, and precedes the z-score operation used to integrate imputed batches.

## Related tools

- **self-consistent expression recovery machine (SERM)** (Pipeline whose final 'Resampling For Patch Consistency' stage applies bilinear resampling to merge the per-ROI histograms into a consistent framework; installable as the Python package 'serm') — https://github.com/xinglab-ai/self-consistent-expression-recovery-machine

## Examples

```
import serm as srm; out_SERM = srm.serm(dataObs, [6100,1250], 0.5, randomize=False)  # ROI-imputed output then resampled for patch consistency
```

## Evaluation signals

- Output matrix has exactly q rows and m columns matching the observed matrix dimensions
- Each interior resampled value equals the four-neighbor diagonal average: s_bar_ij = (1/4)(s_hat_(i-1)(j-1) + s_hat_(i-1)(j+1) + s_hat_(i+1)(j+1) + s_hat_(i+1)(j-1))
- Boundary elements follow the zero-padding rule (out-of-matrix neighbors set to 0)
- Downstream check: imputed data shows similar number of zeros to the reference data and improved Pearson coefficient versus the observed data (e.g., Pearson > 0.9 for most genes at 5% sampling efficiency)
- Batch-integrated results (via z-score operation) pass downstream clustering quality indices (accuracy, NMI, AR, Hubert) comparable to or better than baselines

## Limitations

- Boundary elements are computed under zero padding, so edge values rely on fewer neighbors and may be less stable
- An implicit assumption is that the coverage of each ROI is sufficiently large so that the captured data distribution is representative of the entire matrix; small ROIs undermine patch consistency
- SERM learns the gene expression model from a small fraction of the data, which can cause inaccurate imputation values for small genomic datasets, propagating into the resampled matrix
- Batch effects are not considered explicitly in the formulation; different batches need the z-score integration step before downstream analyses

## Evidence

- [methods] resampling formula: "In the last step, all the ROIs are interpolated using bilinear resampling to achieve the global consistency of the expression values"
- [methods] patch consistency stage: "We now incorporate the histograms of different ROIs into a consistent framework by applying a bilinear resampling."
- [methods] ROI division: "We divide the expression matrix S is divided into N ≥ 4 ROIs, which we denote by Si, i = 1, 2, ⋯ , N."
- [methods] sliding ROI: "The ROI is slided throughout the whole expression matrix and gene imputation is performed at each ROI"
- [methods] batch z-score integration: "The SERM-imputed datasets from different batches are then integrated into a single dataset using a z-score operation for downstream analyses."
- [readme] ROI overlap parameter: "The user can choose any value between 0.01 to 0.99. However, for small overlap, the consecutive windows would be disconnected"
- [intro] scalability rationale: "Because of this divide-and-conquer strategy, SERM is scalable to large datasets and can impute the datasets much faster than other analytical or deep learning-based methods."
- [discussion] small-dataset caveat: "This poses a challenge in dealing with small genomic datasets, as the estimated distribution in SERM may diverge from the"
- [discussion] ROI representativeness assumption: "An implicit assumption in our formulation is that the coverage of ROI is sufficiently large so that the captured data distribution in the ROI is representative of the entire matrix"
