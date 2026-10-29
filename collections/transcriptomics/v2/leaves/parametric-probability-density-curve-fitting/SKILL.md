---
name: parametric-probability-density-curve-fitting
description: Use when a denoised expression matrix (e.g., the SERM autoencoder output
  So) must be summarized as a single parametric probability distribution before histogram-equalization
  imputation, and you need to choose among Gaussian, Rayleigh, and exponential families
  objectively.
when_to_use_negative:
- The data have not been denoised/reconstructed first — fitting PDFs to raw sparse
  scRNA-seq counts with dropouts will model the dropout artifact rather than the underlying
  expression distribution
- Small genomic datasets where the 2000-element autoencoder subset may yield an estimated
  distribution that diverges from the actual data distribution
- The goal is imputation by a non-parametric or deep generative model (e.g., scVI,
  MAGIC) rather than a parametric histogram-equalization pipeline
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: self-consistent expression recovery machine (SERM)
  role: Provides the autoencoder-denoised expression matrix So and implements the
    curve-fitting stage that fits Gaussian, Rayleigh, and exponential PDFs to its
    histogram before histogram-equalization imputation
  repo: https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
- name: SAVER
  role: Baseline imputation method compared against SERM; not part of the curve-fitting
    step itself
- name: MAGIC
  role: Graph-based imputation baseline used for benchmarking against the SERM distribution-learning
    pipeline
  repo: https://github.com/pkathail/magic
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-34595-w
    title: Leveraging data-driven self-consistency for high-fidelity gene expression
      recovery
  source_content_hash: sha256:5d53522600617c180b01e58e85039730549aabfa69e430912e4d9369e1693845
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
  tools:
  - MAGIC
  - SAVER
  - self-consistent expression recovery machine (SERM)
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# parametric-probability-density-curve-fitting

## Summary

Fits candidate probability density functions (Gaussian, Rayleigh, exponential) to the normalized histogram of autoencoder-reconstructed gene expression values, then selects the best-fitting distribution by minimum root mean square error. This is the distribution-learning stage of SERM (Fig. 1 steps 2–3) that supplies the parametric PDF used downstream for histogram-equalization imputation.

## When to use

Use when a denoised expression matrix (e.g., the SERM autoencoder output So) must be summarized as a single parametric probability distribution before histogram-equalization imputation, and you need to choose among Gaussian, Rayleigh, and exponential families objectively.

## When NOT to use

- The data have not been denoised/reconstructed first — fitting PDFs to raw sparse scRNA-seq counts with dropouts will model the dropout artifact rather than the underlying expression distribution
- Small genomic datasets where the 2000-element autoencoder subset may yield an estimated distribution that diverges from the actual data distribution
- The goal is imputation by a non-parametric or deep generative model (e.g., scVI, MAGIC) rather than a parametric histogram-equalization pipeline

## Inputs

- Denoised expression matrix So produced by the SERM three-layer autoencoder (logistic sigmoid encoder, linear decoder, latent dimension P=2)
- data_observed_10perc.mat loaded via scipy.io.loadmat (2000-element random subset fed to the autoencoder)
- Histogram of reconstructed expression values with 100 normalized bins

## Outputs

- Fitted lambda parameter for each candidate PDF (Gaussian, Rayleigh, exponential)
- Root mean square error per candidate distribution
- Name of the selected best-fit distribution and its lambda, e.g., in learned_pdf_parameters

## How to apply

First compute the normalized histogram of the denoised expression values (matrix So) using 100 bins; SERM counts, for each expression value, the number of elements with that value in So. Then fit Gaussian, Rayleigh, and exponential PDFs to the histogram with the Trust-Region algorithm using mean squared error as the cost function, with lambda start/lower/upper bounds of 0.2/0.01/1 for Gaussian and Rayleigh and 5/5/20 for exponential. Select the PDF whose fit yields the minimum root mean square error and record its fitted lambda; this learned distribution f(x) is later used as the target CDF in adaptive histogram equalization over ROIs. Rationale: the autoencoder reconstruction is assumed to approximate the underlying clean expression distribution, so fitting a small parametric family to its histogram gives a compact, reusable distribution model for self-consistent imputation.

## Related tools

- **self-consistent expression recovery machine (SERM)** (Provides the autoencoder-denoised expression matrix So and implements the curve-fitting stage that fits Gaussian, Rayleigh, and exponential PDFs to its histogram before histogram-equalization imputation) — https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
- **SAVER** (Baseline imputation method compared against SERM; not part of the curve-fitting step itself)
- **MAGIC** (Graph-based imputation baseline used for benchmarking against the SERM distribution-learning pipeline) — https://github.com/pkathail/magic

## Examples

```
import scipy.io as sio, serm as srm; dataObs = sio.loadmat('data_observed_10perc.mat')['data_observed']; out_SERM = srm.serm(dataObs, [6100,1250], 0.5, randomize=False)
```

## Evaluation signals

- The output file learned_pdf_parameters exists and contains the three fitted lambda values plus the selected distribution name
- The chosen lambda falls within its fitted bounds: 0.01–1 for Gaussian/Rayleigh, 5–20 for exponential
- The selected distribution is the one with the minimum root mean square error among the three fitted PDFs
- The histogram was normalized and computed with exactly 100 bins over the denoised values in So
- Downstream histogram equalization over ROIs uses the selected f(x) as the target CDF, producing imputed data with improved Pearson coefficient versus the observed data (e.g., >0.9 for most genes at 5% sampling efficiency)

## Limitations

- SERM learns the gene expression model from a small fraction of the data (a 2000-element subset), so on small genomic datasets the estimated distribution may diverge from the actual data distribution, leading to inaccurate imputation values
- An implicit assumption is that the ROI coverage is sufficiently large so that the captured data distribution in the ROI is representative of the entire matrix
- Reference data for evaluation were created by choosing only cells and genes with high expressions, and dropouts were simulated rather than observed experimentally

## Evidence

- [intro] Curve fitting to learn the expression distribution: "It then uses a curve fitting technique to learn a probability distribution from a parametric family to represent the reconstructed expression values"
- [methods] Fitting PDFs after histogram equalization: [evidence span withheld — anchor status: not_found]
- [methods] Histogram computation counts elements per value: "For each expression value, the number of elements with that value in So is counted."
- [readme] SERM first learns the data distribution with a neural network: "Using deep learning, SERM first learns from a subset of the noisy gene expression data to estimate the underlying data distribution."
- [discussion] Small-data distribution divergence caveat: "SERM learns the gene expression model from a small fraction of the data. This poses a challenge in dealing with small genomic datasets, as the estimated distribution in SERM may diverge from the"
