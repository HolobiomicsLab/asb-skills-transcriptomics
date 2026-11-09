---
name: sliding-window-matrix-imputation
description: Use when a dropout-affected gene expression matrix (e.g. 'data_observed_10perc.mat')
  must be imputed after a distribution (Gaussian, Rayleigh, or exponential with decay
  parameter λ) has already been fitted to the reconstructed data.
when_to_use_negative:
- The underlying data distribution has not yet been learned (no fitted PDF/λ available)
  — this stage presupposes the modeling stage output
- The genomic dataset is very small, so the distribution estimated from a small fraction
  of the data may diverge from the actual data distribution
- The data are already a dense, complete matrix without simulated or real dropouts
  — no imputation is needed
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: self-consistent expression recovery machine (SERM)
  role: Implements ROI selection, adaptive histogram equalization using the learned
    PDF, and x/y sliding-window imputation of the expression matrix
  repo: https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-34595-w
    title: Leveraging data-driven self-consistency for high-fidelity gene expression
      recovery
  source_content_hash: sha256:80c1fdf70c21c4f9775bfd5f18e3522116330f5f800dcdc9f5b43c11ecd8e6a7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
  tools:
  - self-consistent expression recovery machine (SERM)
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# sliding-window-matrix-imputation

## Summary

The fixed imputation stage of SERM, in which an ROI is histogram-equalized using a fitted PDF and slid along the x and y axes of the scRNA-seq expression matrix, imputing every window until the full matrix is covered.

## When to use

Use when a dropout-affected gene expression matrix (e.g. 'data_observed_10perc.mat') must be imputed after a distribution (Gaussian, Rayleigh, or exponential with decay parameter λ) has already been fitted to the reconstructed data.

## When NOT to use

- The underlying data distribution has not yet been learned (no fitted PDF/λ available) — this stage presupposes the modeling stage output
- The genomic dataset is very small, so the distribution estimated from a small fraction of the data may diverge from the actual data distribution
- The data are already a dense, complete matrix without simulated or real dropouts — no imputation is needed

## Inputs

- Dropout-affected observed expression matrix loaded from a .mat file (e.g. 'data_observed_10perc.mat' via scipy.io.loadmat; rows are cells, columns are genes)
- Fitted PDF (Gaussian, Rayleigh, or exponential) with decay parameter λ chosen by minimum RMSE in the preceding modeling stage
- ROI size list (window length and height, e.g. [6100, 1250])
- Percent overlap between successive ROIs (between 0.01 and 0.99; 0.5 default)

## Outputs

- Imputed expression matrix Ŝ with the same q×m dimensions as the input
- Window histograms whose equalized CDFs follow the minimum-RMSE fitted distribution's CDF

## How to apply

Divide the expression matrix S into N ≥ 4 rectangular ROIs S_i, using a window length of half the matrix length and window height of half the matrix height (e.g. ROIsize = [6100, 1250] for 12,162 cells × 2,422 genes in the SERM example). For each ROI compute the normalized histogram p_Si(n) = N_Si(n)/N_Si with 100 bins and its CDF F_Si(n), then apply the adaptive equalization transform T_Si(k) = −⌊(1/λ) ln(1 − F_Si(k))⌋, derived from inverse transform sampling, so the transformed CDF approximates the CDF of the fitted PDF. Slide the ROI along the x- and y-axes with the sliding parameters set to the window length and height, equalizing each window position until the entire matrix is covered; the README's percent-overlap parameter controls how much overlap consecutive ROIs share (values 0.25–0.75 work well). The rationale is that equalizing every local window against the globally learned distribution enforces self-consistency of expression values across the matrix, which is computationally cheap compared to deep-learning imputers.

## Related tools

- **self-consistent expression recovery machine (SERM)** (Implements ROI selection, adaptive histogram equalization using the learned PDF, and x/y sliding-window imputation of the expression matrix) — https://github.com/xinglab-ai/self-consistent-expression-recovery-machine

## Examples

```
import scipy.io as sio; import serm as srm; dataObs = sio.loadmat('data_observed_10perc.mat')['data_observed']; out_SERM = srm.serm(dataObs, [6100, 1250], 0.5, randomize=False)
```

## Evaluation signals

- Confirm the imputed matrix Ŝ has the same q×m dimensions as the input expression matrix
- Verify that equalized window histograms approximate the CDF of the minimum-RMSE fitted distribution (Gaussian, Rayleigh, or exponential)
- Check the number of zeros in the imputed data is similar to the number of zeros in the reference (dropout-free) data
- Compare mean Pearson coefficient of imputed vs. observed data against baselines (SERM reports >20% improvement over state-of-the-art in most cases)
- Confirm ROI parameters were respected: matrix divided into N ≥ 4 ROIs, with percent overlap in the 0.01–0.99 range

## Limitations

- SERM learns the gene expression model from a small fraction of the data, so for small genomic datasets the estimated distribution may diverge from the actual distribution, leading to inaccurate imputed values
- An implicit assumption is that ROI coverage is sufficiently large that the captured data distribution in the ROI is representative of the entire matrix
- Batch effects are not considered explicitly in the formulation; a z-score operation is needed when integrating SERM-imputed data across batches

## Evidence

- [intro] ROI selection in workflow: "Next, a rectangular region of interest (ROI) is selected within the expression matrix (Fig. 1-step 4)"
- [intro] Sliding equalization: "The ROI then slides along the x and y direction throughout the expression matrix, and histogram equalization is performed on each ROI."
- [methods] ROI division into N>=4: "The ROI is slided throughout the whole expression matrix and gene imputation is performed at each ROI"
- [readme] README ROI sizing guidance: "Good values for ROI height can be: cell number divided by 4 or 8 and ROI width can be gene number divided by 4 or 8."
