---
name: gene-time-correlation-analysis
description: Use when ranking genes by Pearson correlation with a continuous time
  variable (e.g., a 12-hour C. elegans developmental course) and comparing top-N correlation
  distributions to score denoising recovery.
when_to_use_negative:
- The expression matrix has no continuous, ordered time (or pseudotime) variable —
  per-gene correlation with time is undefined for unordered conditions or cell-type
  labels.
- Only corrupted data with no unperturbed reference is available, since the recovery
  metric requires original-data correlations as the target.
- You need an absolute denoising-quality score with ground truth; the article notes
  that defining a ground truth for real data is challenging, so this comparison is
  qualitative.
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0203
tools:
- name: DCA
  role: Denoises the 80%-dropout-perturbed C. elegans matrix (NB/ZINB loss, 64-32-64
    hidden layers, RMSProp lr 0.001, early stopping after 25 epochs); the strongest-recovering
    method in the correlation-recovery comparison
  repo: https://github.com/theislab/dca
- name: MAGIC
  role: Comparison denoising method (version 0.1.0) run in parallel on the corrupted
    matrix; its gene-time correlation distribution is plotted against DCA's
  repo: https://github.com/pkathail/magic
- name: SAVER
  role: Comparison denoising method (version 0.3.0); contributes one of the five per-method
    correlation distributions
  repo: https://github.com/mohuangx/SAVER
- name: scImpute
  role: Comparison imputation method (version 0.0.5); contributes one of the five
    per-method correlation distributions
  repo: https://github.com/Vivianstats/scImpute
- name: Scanpy
  role: Python API exposing the 'dca' preprocessing method so the denoising step runs
    inside a Scanpy/anndata workflow
- name: anndata
  role: h5ad-formatted container used to load and pass the count matrices into the
    correlation-recovery pipeline
  repo: https://github.com/theislab/anndata
- name: pandas
  role: Loads the count matrix and computes per-gene Pearson correlations with time
    (e.g., DataFrame.corrwith) for top-500 selection
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-018-07931-2
    title: Single-cell RNA-seq denoising using a deep count autoencoder
  source_content_hash: sha256:16c7d6fea5d33477295af88cb7a7589ca7da61605ad364be57e121d783743f70
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/dca
  tools:
  - DCA
  - MAGIC
  - SAVER
  - Scanpy
  - anndata
  - pandas
  - scImpute
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# gene-time-correlation-analysis

## Summary

Computes per-gene Pearson correlations between expression and developmental time on the Francesconi et al. C. elegans bulk time-course (206 samples spanning 12 hours), and uses the correlation distribution over the top 500 time-correlated genes as a recovery metric to benchmark denoising methods (DCA, MAGIC, SAVER, scImpute) against a dropout-perturbed (80% zeros) baseline.

## When to use

Use when ranking genes by Pearson correlation with a continuous time variable (e.g., a 12-hour C. elegans developmental course) and comparing top-N correlation distributions to score denoising recovery.

## When NOT to use

- The expression matrix has no continuous, ordered time (or pseudotime) variable — per-gene correlation with time is undefined for unordered conditions or cell-type labels.
- Only corrupted data with no unperturbed reference is available, since the recovery metric requires original-data correlations as the target.
- You need an absolute denoising-quality score with ground truth; the article notes that defining a ground truth for real data is challenging, so this comparison is qualitative.

## Inputs

- Francesconi et al. bulk C. elegans gene expression count matrix (206 samples, 12-hour time-course)
- Per-sample developmental time values for the 206 samples
- Optional: dropout-perturbed matrix (80% zeros) plus its DCA/MAGIC/SAVER/scImpute denoised counterparts, gene-aligned to the original

## Outputs

- Per-gene Pearson correlation coefficients between expression and time
- Ranked list of the top 500 time-correlated genes selected from the original unperturbed data
- Per-method gene-time correlation distributions (noisy, DCA, MAGIC, SAVER, scImpute) over the fixed top-500 gene set
- Fig. 4d-style recovery comparison plot

## How to apply

Load the Francesconi et al. bulk C. elegans gene expression count matrix (206 samples covering a 12-hour time-course) with pandas/anndata and attach the per-sample time values. Compute the Pearson correlation between each gene's expression profile and time on the original data, rank genes, and fix the top 500 time-correlated genes as the evaluation set. Optionally, perturb the matrix with simulated dropout (subtract gene-wise exponentially distributed values until 80% of entries become zeros) and produce denoised copies with DCA (Keras/TensorFlow, NB/ZINB loss, default 64-32-64 hidden layers, RMSProp learning rate 0.001, early stopping after 25 epochs), MAGIC 0.1.0, SAVER 0.3.0 and scImpute 0.0.5. Recompute gene-time Pearson correlations on every noisy/denoised matrix, restricting all methods to the same fixed top-500 gene set so the distributions are directly comparable. Plot the five correlation distributions (noisy, DCA, MAGIC, SAVER, scImpute) as the Fig. 4d-style recovery comparison; recovery is judged qualitatively by how far each method's distribution shifts back toward the original correlations, with DCA strongest and the paper reporting no numeric threshold.

## Related tools

- **DCA** (Denoises the 80%-dropout-perturbed C. elegans matrix (NB/ZINB loss, 64-32-64 hidden layers, RMSProp lr 0.001, early stopping after 25 epochs); the strongest-recovering method in the correlation-recovery comparison) — https://github.com/theislab/dca
- **MAGIC** (Comparison denoising method (version 0.1.0) run in parallel on the corrupted matrix; its gene-time correlation distribution is plotted against DCA's) — https://github.com/pkathail/magic
- **SAVER** (Comparison denoising method (version 0.3.0); contributes one of the five per-method correlation distributions) — https://github.com/mohuangx/SAVER
- **scImpute** (Comparison imputation method (version 0.0.5); contributes one of the five per-method correlation distributions) — https://github.com/Vivianstats/scImpute
- **Scanpy** (Python API exposing the 'dca' preprocessing method so the denoising step runs inside a Scanpy/anndata workflow)
- **anndata** (h5ad-formatted container used to load and pass the count matrices into the correlation-recovery pipeline) — https://github.com/theislab/anndata
- **pandas** (Loads the count matrix and computes per-gene Pearson correlations with time (e.g., DataFrame.corrwith) for top-500 selection)

## Examples

```
gene_time = expr.corrwith(time, axis=1, method='pearson'); top500 = gene_time.nlargest(500).index
```

## Evaluation signals

- All five correlation distributions (noisy, DCA, MAGIC, SAVER, scImpute) are generated over the same fixed top-500 gene set.
- The top-500 evaluation set is derived from the original unperturbed data and held identical across all five conditions.
- The perturbed matrix is verified to contain ~80% zero entries before denoising.
- Noisy-baseline correlations are strongly degraded relative to the original; each method is judged by how far its distribution shifts back toward the original correlations.
- DCA's distribution shows the strongest recovery among the four denoising methods, qualitatively matching published Fig. 4d (the paper reports no numeric threshold).

## Limitations

- The paper reports no numeric threshold for the Fig. 4d comparison, so acceptance is qualitative.
- The evaluation of denoising is difficult because the definition of a ground truth can be challenging for real data.
- Dropout is simulated on bulk data by gene-wise exponential subtraction (80% zeros), not real scRNA-seq sparsity; recovery rankings may not transfer to true single-cell noise.
- Hyperparameters were not fine-tuned for any of the analyses in the manuscript, so per-method settings may not be optimal.

## Evidence

- [discussion] Dataset description: "Francesconi et al36. data set contained 206 samples covering a 12-hour time-course."
- [discussion] Denoised output definition: "The denoised matrix is generated by replacing the original count values with the mean of the negative binomial component"
- [discussion] Training schedule: "We used RMSProp for optimization with learning rate 0.001. Learning rate is multiplied by 0.1 if validation loss does not improve for 20 epochs."
- [discussion] Evaluation difficulty: "The evaluation of denoising is difficult because the definition of a ground truth can be challenging for real data."
- [intro] DCA default architecture: "The deep learning framework (by default three hidden layers with 64, 32, 64 neurons) of DCA enables the capturing of the complexity and non-linearity in scRNA-seq data."
- [readme] MAGIC method description: [evidence span withheld — anchor status: external_doc]
- [readme] SAVER method description: [evidence span withheld — anchor status: external_doc]
