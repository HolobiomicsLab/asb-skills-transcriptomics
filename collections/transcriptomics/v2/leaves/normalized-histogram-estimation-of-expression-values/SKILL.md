---
name: normalized-histogram-estimation-of-expression-values
description: Use when a denoised/reconstructed expression matrix is available and
  you need a parametric distribution fitted to the expression values before histogram-equalization
  imputation (SERM Fig. 1 steps 2-3).
when_to_use_negative:
- Input is the raw sparse observed data rather than the autoencoder-reconstructed
  matrix So — SERM fits PDFs only to the denoised reconstruction (Fig. 1 steps 1-3).
- 'Small genomic datasets where a small fraction of the data is used to learn the
  distribution: the estimated distribution may diverge from the actual data distribution,
  leading to inaccurate imputation values.'
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3308
tools:
- name: self-consistent expression recovery machine (SERM)
  role: Framework whose curve-fitting stage fits Gaussian, Rayleigh, and exponential
    PDFs to the normalized histogram of the autoencoder-reconstructed values
  repo: https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-34595-w
    title: Leveraging data-driven self-consistency for high-fidelity gene expression
      recovery
  source_content_hash: sha256:323e774ba8fdccc270ad25842e9f490f275a9160a1ef77e2f53c13761db7216d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
  tools:
  - self-consistent expression recovery machine (SERM)
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# normalized-histogram-estimation-of-expression-values

## Summary

A SERM modeling-stage skill that converts autoencoder-denoised expression values (matrix So) into a normalized histogram and fits parametric PDFs (Gaussian, Rayleigh, exponential) to learn the probability distribution that best describes the reconstructed scRNA-seq expression values.

## When to use

Use when a denoised/reconstructed expression matrix is available and you need a parametric distribution fitted to the expression values before histogram-equalization imputation (SERM Fig. 1 steps 2-3).

## When NOT to use

- Input is the raw sparse observed data rather than the autoencoder-reconstructed matrix So — SERM fits PDFs only to the denoised reconstruction (Fig. 1 steps 1-3).
- Small genomic datasets where a small fraction of the data is used to learn the distribution: the estimated distribution may diverge from the actual data distribution, leading to inaccurate imputation values.

## Inputs

- Autoencoder-denoised expression matrix So (from data_observed_10perc.mat loaded via scipy.io.loadmat)
- Number of histogram bins (100 in the SERM workflow)
- PDF candidate family and lambda bounds (Gaussian/Rayleigh: 0.01-1; exponential: 5-20)

## Outputs

- Normalized histogram pSo of the denoised expression values (100 bins)
- Fitted lambda values for Gaussian, Rayleigh, and exponential PDFs
- Name of the selected distribution (minimum RMSE) and its fitted lambda, e.g. written to learned_pdf_parameters

## How to apply

Take the denoised matrix So produced by the three-layer autoencoder (logistic sigmoid encoder, linear decoder, latent dimension P=2, L2/sparse regularizers 0.05/0.9, learning rate 1e-5, max 20 epochs, batch size 64, adam) trained on a randomly selected 2000-element subset of the observed data. Count, for each expression value, the number of elements with that value in So to build the histogram, then normalize it using 100 bins. Fit Gaussian, Rayleigh, and exponential PDFs to the normalized histogram pSo with the Trust-Region algorithm using mean squared error as the cost, constraining lambda to 0.2 start (0.01-1 bounds) for Gaussian/Rayleigh and 5 start (5-20 bounds) for exponential. Select the PDF with the minimum root mean square error and record its fitted lambda; this learned distribution is later used for histogram equalization within ROIs (e.g. 6100x1250 ROIs with 50% overlap in the Baryawno et al. bone-marrow stroma example).

## Related tools

- **self-consistent expression recovery machine (SERM)** (Framework whose curve-fitting stage fits Gaussian, Rayleigh, and exponential PDFs to the normalized histogram of the autoencoder-reconstructed values) — https://github.com/xinglab-ai/self-consistent-expression-recovery-machine

## Examples

```
import serm as srm; a=sio.loadmat('data_observed_10perc.mat'); dataObs=a['data_observed']; out_SERM=srm.serm(dataObs,[6100,1250],0.5,randomize=False)
```

## Evaluation signals

- Output file learned_pdf_parameters exists and contains the three fitted lambda values plus the selected distribution name.
- The chosen lambda falls within its fitted bounds: 0.01-1 for Gaussian/Rayleigh, 5-20 for exponential.
- The selected PDF is the one with the minimum root mean square error among the three fitted candidates.
- Histogram of So is normalized and computed with 100 bins as in the SERM workflow.

## Limitations

- SERM learns the gene expression model from a small fraction of the data, so for small genomic datasets the estimated distribution may diverge from the actual data distribution, leading to inaccurate imputation values.
- The formulation implicitly assumes ROI/data coverage is sufficiently large so the captured distribution is representative of the entire matrix.
- Reference data for evaluation was created by choosing only cells and genes with high expressions, and dropouts were simulated rather than observed experimentally.

## Evidence

- [methods] For each expression value, the number of elements with that value in So is counted.: "For each expression value, the number of elements with that value in So is counted."
- [methods] After applying the histogram equalization, we fit different probability distribution functions (PDF) to pSo.: [evidence span withheld — anchor status: not_found]
- [intro] It then uses a curve fitting technique to learn a probability distribution from a parametric family to represent the reconstructed expression values: "It then uses a curve fitting technique to learn a probability distribution from a parametric family to represent the reconstructed expression values"
- [readme] The following two code lines read the observed data for 10% sampling efficiency: "# The following two code lines read the observed data for 10% sampling efficiency"
