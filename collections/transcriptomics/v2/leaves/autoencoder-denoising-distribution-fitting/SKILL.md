---
name: autoencoder-denoising-distribution-fitting
description: Use when a scRNA-seq count matrix (cells x genes) is sparse due to dropout/low
  capture efficiency and you need denoised imputed expression values without training
  method-specific deep models per dataset.
when_to_use_negative:
- 'Small genomic datasets: the paper notes SERM learns from a small fraction of the
  data, so the estimated distribution may diverge from the actual data distribution
  and yield inaccurate imputations'
- Cases where explicit batch-effect correction is required — batch effects are not
  considered in the SERM formulation
- Datasets where ROI coverage is too small for the captured ROI distribution to be
  representative of the entire matrix
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: self-consistent expression recovery machine (SERM)
  role: Implements the autoencoder denoising, distribution curve fitting, ROI histogram
    equalization, and bilinear resampling pipeline (pip install serm)
  repo: https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
- name: DeepImpute
  role: Baseline deep-learning imputation method whose autoencoder-based approach
    the distribution-fitting strategy is benchmarked against on the same observed
    matrices
  repo: https://github.com/lanagarmire/deepimpute
- name: SAUCIE
  role: Tensorflow sparse-autoencoder baseline (clustering, imputing, embedding) used
    for comparison against the SERM autoencoder-denoising approach
  repo: https://github.com/KrishnaswamyLab/SAUCIE
- name: DCA
  role: Autoencoder-based denoising baseline for imputation comparison
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-34595-w
    title: Leveraging data-driven self-consistency for high-fidelity gene expression
      recovery
  source_content_hash: sha256:3a1eaa65db19ad81d1d11acd127404a4514faaa2f6e48e192313faad2e7f3f17
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 18
---

# autoencoder-denoising-distribution-fitting

## Summary

Core SERM mechanism from Islam et al. (Nat Commun 2022): a deep autoencoder first learns the underlying scRNA-seq expression distribution from the sparse observed matrix, then a fitted probability distribution is used for histogram-based imputation of the gene expression matrix.

## When to use

Use when a scRNA-seq count matrix (cells x genes) is sparse due to dropout/low capture efficiency and you need denoised imputed expression values without training method-specific deep models per dataset.

## When NOT to use

- Small genomic datasets: the paper notes SERM learns from a small fraction of the data, so the estimated distribution may diverge from the actual data distribution and yield inaccurate imputations
- Cases where explicit batch-effect correction is required — batch effects are not considered in the SERM formulation
- Datasets where ROI coverage is too small for the captured ROI distribution to be representative of the entire matrix

## Inputs

- Observed (dropout-affected) gene expression matrix as a .mat file, e.g. data_observed_10perc.mat with key 'data_observed' (rows = cells, columns = genes)
- ROI size parameters (e.g., ROIsize = [6100, 1250])
- Percent overlap between successive ROIs (percOL, e.g., 0.5)

## Outputs

- Imputed gene expression matrix returned by srm.serm (same cells x genes shape as the observed matrix)
- Downstream per-cell Pearson correlation coefficients against a dropout-free reference (e.g., data_reference.mat) for benchmarking

## How to apply

Run the SERM autoencoder (encoder + decoder) on the observed expression matrix to learn the underlying data distribution and reconstruct denoised expression values via deep-learning optimization. Fit probability distribution functions (PDFs) from a parametric family to the histogram of the reconstructed values using curve fitting. Divide the matrix into N >= 4 rectangular regions of interest (e.g., ROIsize = [6100, 1250] for the 12,162-cell x 2,422-gene cellular taxonomy matrix, giving a 2x2 tiling), perform adaptive histogram equalization within each ROI using the fitted distribution, slide the ROI over the matrix (percent overlap percOL, with 0.25-0.75 working well; default 0.5), and finally bilinearly resample all ROIs for global consistency. Set random.seed(0) and randomize=False for reproducibility.

## Related tools

- **self-consistent expression recovery machine (SERM)** (Implements the autoencoder denoising, distribution curve fitting, ROI histogram equalization, and bilinear resampling pipeline (pip install serm)) — https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
- **DeepImpute** (Baseline deep-learning imputation method whose autoencoder-based approach the distribution-fitting strategy is benchmarked against on the same observed matrices) — https://github.com/lanagarmire/deepimpute
- **SAUCIE** (Tensorflow sparse-autoencoder baseline (clustering, imputing, embedding) used for comparison against the SERM autoencoder-denoising approach) — https://github.com/KrishnaswamyLab/SAUCIE
- **DCA** (Autoencoder-based denoising baseline for imputation comparison)

## Examples

```
import serm as srm; import random; random.seed(0); out_SERM = srm.serm(dataObs, [6100, 1250], 0.5, randomize=False)
```

## Evaluation signals

- Per-cell Pearson coefficient between imputed and dropout-free reference expression vectors should exceed that of the observed data by >20% in most cases (e.g., 100 x (rho_method - rho_observed)/rho_observed)
- At 5% sampling efficiency, Pearson coefficients above 0.9 are expected for most genes (versus <= 0.4 for most other methods)
- At 0.1% downsampling, recovered Pearson coefficients should fall in the ~0.15-0.65 range and beat other imputation methods
- Number of zeros in the imputed matrix should be similar to that of the dropout-free reference, indicating accurate zero imputation
- Runtime should be at least ~10x faster than other analytical techniques for the same matrix

## Limitations

- SERM learns the gene expression model from a small fraction of the data, which is problematic for small genomic datasets where the estimated distribution may diverge from the actual distribution
- Batch effects are not considered explicitly in the SERM formulation (though it is generally less susceptible to differing technologies/protocols)
- Performance implicitly assumes the ROI coverage is sufficiently large that the captured distribution is representative of the entire matrix
- Reference/observed evaluation data simulate dropouts (Poisson sampling with Gamma-drawn cell-specific efficiency, e.g., Gamma(10, 100) for 10% efficiency) rather than using experimentally observed dropouts

## Evidence

- [methods] The deep learning architecture (autoencoder) used in SERM consists of two parts, an encoder and a decoder.: "The deep learning architecture (autoencoder) used in SERM consists of two parts, an encoder and a decoder."
- [intro] It then uses a curve fitting technique to learn a probability distribution from a parametric family to represent the reconstructed expression values: "It then uses a curve fitting technique to learn a probability distribution from a parametric family to represent the reconstructed expression values"
- [intro] imputation is performed based on histogram equalization using the learned probability distribution: "imputation is performed based on histogram equalization using the learned probability distribution (Fig. 1-step 5)"
- [readme] The user can choose any value between 0.01 to 0.99. However, for small overlap... A value between 0.25 or 0.75 works well with SERM.: "A value between 0.25 or 0.75 works well with SERM."
