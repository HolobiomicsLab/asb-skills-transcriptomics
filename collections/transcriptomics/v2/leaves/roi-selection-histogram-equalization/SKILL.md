---
name: roi-selection-histogram-equalization
description: Use when you have a sparse, dropout-affected scRNA-seq expression matrix
  plus a fitted PDF from a prior distribution-learning step, and you need to impute
  missing values by analytically enforcing self-consistency via ROI-based histogram
  equalization.
when_to_use_negative:
- 'The dataset is small: SERM learns the distribution from a small fraction of the
  data, so the estimated distribution may diverge from the actual data distribution
  and yield inaccurate imputations.'
- 'You need explicit batch-effect correction: batch effects are not considered explicitly
  in the SERM formulation (only a z-score integration of batches is applied downstream).'
- You only have raw unfiltered counts and no fitted distribution from the modeling
  stage — the equalization transform T_Si(k) requires the fitted PDF's decay parameter
  λ.
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3511
- http://edamontology.org/topic_2229
tools:
- name: self-consistent expression recovery machine (SERM)
  role: Implements ROI selection, adaptive histogram equalization, and sliding-window
    imputation of the expression matrix; invoked as srm.serm(dataObs, ROIsize, percOL,
    randomize)
  repo: https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
- name: SciPy
  role: Loads the observed dropout-affected expression matrix from .mat files (scipy.io.loadmat)
    before running SERM
- name: MAGIC
  role: Baseline imputation method for comparison against the ROI-based SERM imputation
  repo: https://github.com/pkathail/magic
- name: mcImpute
  role: Baseline matrix-completion imputation method for comparison against SERM
  repo: https://github.com/aanchalMongia/McImpute_scRNAseq
- name: DeepImpute
  role: Baseline deep-learning imputation method for comparison against SERM
  repo: https://github.com/lanagarmire/deepimpute
- name: SAUCIE
  role: Baseline autoencoder imputation method for comparison against SERM
  repo: https://github.com/KrishnaswamyLab/SAUCIE
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-34595-w
    title: Leveraging data-driven self-consistency for high-fidelity gene expression
      recovery
  source_content_hash: sha256:779aa4c6cac15cfc7342a4df38580a4a5bd2b925accd173216bf8398330b0cf9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
  tools:
  - AutoImpute
  - DCA
  - DeepImpute
  - MAGIC
  - SAUCIE
  - SAVER
  - SERM
  - SciPy
  - mcImpute
  - scScope
  - scVI
  - self-consistent expression recovery machine (SERM)
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3511
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 22
---

# roi-selection-histogram-equalization

## Summary

SERM's fixed imputation stage: divide the dropout-affected scRNA-seq expression matrix into overlapping rectangular ROIs and impute each window by adaptive histogram equalization whose target CDF is the parametric distribution (Gaussian, Rayleigh, or exponential with decay parameter λ) fitted in the modeling stage.

## When to use

Use when you have a sparse, dropout-affected scRNA-seq expression matrix plus a fitted PDF from a prior distribution-learning step, and you need to impute missing values by analytically enforcing self-consistency via ROI-based histogram equalization.

## When NOT to use

- The dataset is small: SERM learns the distribution from a small fraction of the data, so the estimated distribution may diverge from the actual data distribution and yield inaccurate imputations.
- You need explicit batch-effect correction: batch effects are not considered explicitly in the SERM formulation (only a z-score integration of batches is applied downstream).
- You only have raw unfiltered counts and no fitted distribution from the modeling stage — the equalization transform T_Si(k) requires the fitted PDF's decay parameter λ.

## Inputs

- Dropout-affected observed expression matrix (cells x genes, e.g. data_observed_10perc.mat via scipy.io.loadmat)
- Fitted probability distribution (Gaussian, Rayleigh, or exponential with decay parameter λ selected by minimum RMSE) from SERM's distribution-learning stage
- ROI size (e.g. [6100, 1250]) and percent overlap percOL (0.01–0.99, 0.25–0.75 recommended, default 0.5)

## Outputs

- Imputed expression matrix Ŝ with the same q x m dimensions as the input
- Per-window equalized histograms whose CDFs follow the fitted PDF's CDF

## How to apply

Divide the observed expression matrix S (cells x genes, e.g. 12,162 x 2,422 loaded from data_observed_10perc.mat via scipy.io.loadmat) into N >= 4 rectangular ROIs S_i, using a window length of half the matrix length and height of half the matrix height (e.g. ROIsize=[6100, 1250] for a 2x2 tiling). For each ROI, compute the normalized histogram p_Si(n) = N_Si(n)/N_Si with 100 bins and its CDF F_Si(n). Apply the adaptive equalization transform T_Si(k) = −⌊(1/λ) ln(1 − F_Si(k))⌋, derived from inverse transform sampling, so the transformed CDF approximates the CDF of the fitted distribution (the Gaussian, Rayleigh, or exponential PDF chosen by minimum RMSE). Slide the ROI along the x- and y-axes with sliding parameters equal to the window length and height, imputing values at each window position until the entire matrix is covered; then reconcile overlapping ROIs with bilinear resampling (percOL, e.g. 0.5, controls the overlap so consecutive windows stay connected).

## Related tools

- **self-consistent expression recovery machine (SERM)** (Implements ROI selection, adaptive histogram equalization, and sliding-window imputation of the expression matrix; invoked as srm.serm(dataObs, ROIsize, percOL, randomize)) — https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
- **SciPy** (Loads the observed dropout-affected expression matrix from .mat files (scipy.io.loadmat) before running SERM)
- **MAGIC** (Baseline imputation method for comparison against the ROI-based SERM imputation) — https://github.com/pkathail/magic
- **mcImpute** (Baseline matrix-completion imputation method for comparison against SERM) — https://github.com/aanchalMongia/McImpute_scRNAseq
- **DeepImpute** (Baseline deep-learning imputation method for comparison against SERM) — https://github.com/lanagarmire/deepimpute
- **SAUCIE** (Baseline autoencoder imputation method for comparison against SERM) — https://github.com/KrishnaswamyLab/SAUCIE

## Examples

```
import scipy.io as sio, serm as srm, random; dataObs = sio.loadmat('data_observed_10perc.mat')['data_observed']; random.seed(0); out_SERM = srm.serm(dataObs, ROIsize=[6100,1250], percOL=0.5, randomize=False)
```

## Evaluation signals

- Imputed matrix Ŝ has the same q x m dimensions as the input expression matrix.
- Equalized window histograms follow the CDF of the fitted minimum-RMSE distribution (Gaussian, Rayleigh, or exponential).
- Percent improvement in mean Pearson coefficient of imputed vs. observed data exceeds 20% in most cases against the eight baselines (MAGIC, mcImpute, DeepImpute, SAUCIE, AutoImpute, DCA, scScope, scVI/SAVER).
- At 5% sampling efficiency, per-gene Pearson coefficients exceed 0.9 for most genes; at 0.1% downsampling, Pearson coefficients of 0.15–0.65 recover the original values.
- Number of zeros in the SERM-imputed data is similar to that in the dropout-free reference data.

## Limitations

- SERM learns the gene expression model from a small fraction of the data, which can cause the estimated distribution to diverge from the actual distribution on small genomic datasets.
- An implicit assumption is that ROI coverage is sufficiently large that the captured data distribution is representative of the entire matrix.
- Batch effects are not considered explicitly in the SERM formulation.
- Reference/observed data for evaluation were simulated (Poisson sampling with Gamma-drawn cell-specific efficiency, e.g. Gamma(10, 100) for 10%), not experimentally observed dropouts.

## Evidence

- [methods] ROI divide and equalization: "We divide the expression matrix S is divided into N ≥ 4 ROIs, which we denote by Si, i = 1, 2, ⋯ , N."
- [methods] adaptive equalization goal: "The goal of the histogram adaptive equalization is to find a transformation for the expression values such that the transformed CDF is approximated by the CDF of f(x)"
- [methods] sliding window: "The ROI is slided throughout the whole expression matrix and gene imputation is performed at each ROI"
- [intro] overview workflow: "Next, a rectangular region of interest (ROI) is selected within the expression matrix (Fig. 1-step 4)"
- [readme] README invocation parameters: "out_SERM = srm.serm(dataObs,ROIsize,percOL,randomize=False) # If you want to randomize the rows and columns"
- [readme] README ROI guidance: "Good values for ROI height can be: cell number divided by 4 or 8 and ROI width can be gene number divided by 4 or 8."
