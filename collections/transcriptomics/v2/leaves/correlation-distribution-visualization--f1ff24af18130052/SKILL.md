---
name: correlation-distribution-visualization
description: Use when comparing denoising/imputation methods on a time-course expression
  matrix and you need a distribution-level readout of recovery for genes most strongly
  correlated with time, rather than a single aggregate score.
when_to_use_negative:
- When you need a numeric, threshold-based ranking of methods — the paper explicitly
  reports no numeric threshold for this comparison, so it is a qualitative recovery
  check
- When the dataset has no continuous covariate (e.g., time) correlated with gene expression;
  the top-500 gene selection and correlation recovery depend on it
- When ground-truth recovery must be defined for real data — the article notes that
  defining ground truth for real data is challenging and evaluation of denoising is
  difficult
edam_operation: http://edamontology.org/operation_0531
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: DCA (deep count autoencoder)
  role: Produces the denoised expression matrix whose recovered time-correlations
    are visualized; implemented in Python 3 with Keras and TensorFlow backend
  repo: https://github.com/theislab/dca
- name: MAGIC
  role: Comparison denoising method (version 0.1.0) generating a denoised matrix for
    the correlation-distribution comparison
  repo: https://github.com/pkathail/magic
- name: SAVER
  role: Comparison imputation method (version 0.3.0) generating a denoised matrix
    for the correlation-distribution comparison
  repo: https://github.com/mohuangx/SAVER
- name: scImpute
  role: Comparison imputation method (version 0.0.5) generating a denoised matrix
    for the correlation-distribution comparison
  repo: https://github.com/Vivianstats/scImpute
- name: Scanpy
  role: Python analysis framework compatible with DCA's API, usable for handling the
    expression matrices during correlation computation and plotting
  repo: https://github.com/theislab/dca
- name: anndata
  role: h5ad-formatted HDF5 container used to hold the count matrices processed in
    this recovery analysis
  repo: https://github.com/theislab/anndata
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-018-07931-2
    title: Single-cell RNA-seq denoising using a deep count autoencoder
  source_content_hash: sha256:f0139887ddc8d829ba6c845af72887e026799f749184e6a97b0f0e221a432d12
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/dca
  tools:
  - DCA (deep count autoencoder)
  - MAGIC
  - SAVER
  - Scanpy
  - anndata
  - scImpute
  edam_operation: http://edamontology.org/operation_0531
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# correlation-distribution-visualization

## Summary

Quantifies gene-expression recovery after denoising by computing Pearson correlations between each gene's expression and developmental time, restricting to the top 500 time-correlated genes, and plotting the per-method correlation distributions (noisy vs. DCA/MAGIC/SAVER/scImpute) as a Fig. 4d-style recovery comparison.

## When to use

Use when comparing denoising/imputation methods on a time-course expression matrix and you need a distribution-level readout of recovery for genes most strongly correlated with time, rather than a single aggregate score.

## When NOT to use

- When you need a numeric, threshold-based ranking of methods — the paper explicitly reports no numeric threshold for this comparison, so it is a qualitative recovery check
- When the dataset has no continuous covariate (e.g., time) correlated with gene expression; the top-500 gene selection and correlation recovery depend on it
- When ground-truth recovery must be defined for real data — the article notes that defining ground truth for real data is challenging and evaluation of denoising is difficult

## Inputs

- Original C. elegans bulk gene expression count matrix (Francesconi et al., 206 samples, 12-hour time-course) with sample time metadata
- Denoised count matrices produced by DCA, MAGIC (v0.1.0), SAVER (v0.3.0), and scImpute (v0.0.5)
- Noise-perturbed version of the same matrix with 80% zero entries (dropout baseline)

## Outputs

- Per-gene Pearson correlation values (gene expression vs. time) for each condition, restricted to the top 500 time-correlated genes
- A distribution plot (Fig. 4d-style) comparing Pearson correlation distributions across noisy, DCA, MAGIC, SAVER, and scImpute conditions

## How to apply

Load the Francesconi et al. C. elegans bulk count matrix (206 samples over a 12-hour time-course) and compute Pearson correlations between each gene's expression and time in the original data, keeping the top 500 most time-correlated genes. After perturbing the matrix with dropout (80% zeros) and denoising with each method (DCA with negative binomial/ZINB loss, MAGIC v0.1.0, SAVER v0.3.0, scImpute v0.0.5), recompute the time-correlations on each denoised matrix restricted to those same 500 genes. Plot the resulting Pearson correlation distributions side by side for all five conditions (noisy, DCA, MAGIC, SAVER, scImpute) so the shifted/tightened distributions reveal which method best restores the original time structure. Validation is qualitative per the paper: the plot must include all five conditions and the best method's distribution should show recovery of time-correlated genes relative to the noisy baseline; the paper reports no numeric threshold for this comparison.

## Related tools

- **DCA (deep count autoencoder)** (Produces the denoised expression matrix whose recovered time-correlations are visualized; implemented in Python 3 with Keras and TensorFlow backend) — https://github.com/theislab/dca
- **MAGIC** (Comparison denoising method (version 0.1.0) generating a denoised matrix for the correlation-distribution comparison) — https://github.com/pkathail/magic
- **SAVER** (Comparison imputation method (version 0.3.0) generating a denoised matrix for the correlation-distribution comparison) — https://github.com/mohuangx/SAVER
- **scImpute** (Comparison imputation method (version 0.0.5) generating a denoised matrix for the correlation-distribution comparison) — https://github.com/Vivianstats/scImpute
- **Scanpy** (Python analysis framework compatible with DCA's API, usable for handling the expression matrices during correlation computation and plotting) — https://github.com/theislab/dca
- **anndata** (h5ad-formatted HDF5 container used to hold the count matrices processed in this recovery analysis) — https://github.com/theislab/anndata

## Examples

```
import pandas as pd; corr_noisy = pd.DataFrame(noisy, index=genes, columns=times).T.corrwith(times)  # then .corr() distributions per method on top-500 time-correlated genes
```

## Evaluation signals

- The correlation distribution plot is generated for all five conditions: noisy, DCA, MAGIC, SAVER, and scImpute
- The gene set analyzed is exactly the top 500 genes most correlated with developmental time in the original (unperturbed) data
- The best-performing method's correlation distribution shows recovery of time-correlated genes relative to the noisy baseline, qualitatively matching published Fig. 4d
- Per-gene Pearson correlation values are recomputed identically (expression vs. time) across all conditions so distributions are comparable

## Limitations

- The paper reports no numeric threshold for this comparison — validation is qualitative acceptance of the distribution plot
- Evaluation of denoising is difficult because the definition of a ground truth can be challenging for real data
- In general, it may be difficult to determine when denoising improves scRNA-seq data

## Evidence

- [other] Pearson correlation distributions as Fig. 4d recovery comparison: [evidence span withheld — anchor status: not_found]
- [other] restrict to top 500 time-correlated genes: [evidence span withheld — anchor status: not_found]
- [discussion] ground truth challenging for real data: "The evaluation of denoising is difficult because the definition of a ground truth can be challenging for real data."
- [discussion] difficult to determine when denoising improves data: "Note that in general, it may be difficult to determine when denoising improves scRNA-seq data."
- [intro] denoised output for each gene and cell entry: "MAGIC20 and SAVER21, on the other hand, denoise single-cell gene expression data and generate a denoised output for each gene and cell entry"
