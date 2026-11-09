---
name: single-cell-dropout-imputation
description: Use when a scRNA-seq count matrix has extensive simulated or observed
  dropouts (e.g. 0.1%–10% sampling efficiency) and accurate imputation is needed before
  clustering, visualization, or pseudotime analysis.
when_to_use_negative:
- 'Small genomic datasets: SERM learns the distribution from a small fraction of the
  data, so the estimated distribution may diverge from the actual distribution and
  produce inaccurate imputation values'
- Reference/observed pairs where dropouts were experimentally observed rather than
  simulated — the benchmark design (reference filtered to high-expression cells/genes,
  dropouts simulated) may not match such data
- Batch-level analyses requiring explicit batch-effect modeling — batch effects are
  not considered explicitly in the SERM formulation (only z-score integration post
  hoc)
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: self-consistent expression recovery machine (SERM)
  role: 'Primary imputation tool: autoencoder distribution learning + adaptive histogram
    equalization over sliding ROIs to recover dropout values'
  repo: https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
- name: MAGIC
  role: Competing Markov affinity-based graph imputation method run with default configurations
    on the same observed matrices for benchmarking
  repo: https://github.com/pkathail/magic
- name: mcImpute
  role: Competing matrix-completion-based imputation method for benchmarking against
    SERM
  repo: https://github.com/aanchalMongia/McImpute_scRNAseq
- name: DeepImpute
  role: Competing deep learning imputation method for benchmarking against SERM
  repo: https://github.com/lanagarmire/deepimpute
- name: SAUCIE
  role: Competing sparse-autoencoder imputation method for benchmarking against SERM
  repo: https://github.com/KrishnaswamyLab/SAUCIE
- name: PHATE
  role: Visualization of reference, observed, and imputed data (2 dims, 5 nearest
    neighbors, alpha = 40, metric MDS, Euclidean, gamma = 0.5; 100 PCs zebrafish /
    50 PCs EB)
provenance:
  source_task_ids:
  - task_006
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-34595-w
    title: Leveraging data-driven self-consistency for high-fidelity gene expression
      recovery
  source_content_hash: sha256:1ecc412e91b049bbf7b3554f330debaf9ed92b96e5d279272fec5b4495813c6d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
  tools:
  - DeepImpute
  - MAGIC
  - PHATE
  - SAUCIE
  - Slingshot
  - TSCAN
  - mcImpute
  - self-consistent expression recovery machine (SERM)
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# single-cell-dropout-imputation

## Summary

Recovers zeroed-out (dropout) gene expression values in sparse single-cell RNA-seq matrices by learning the underlying data distribution (autoencoder + parametric PDF curve fitting) and imposing self-consistency via adaptive histogram equalization over sliding ROIs (SERM), yielding imputed matrices that improve downstream clustering and trajectory inference.

## When to use

Use when a scRNA-seq count matrix has extensive simulated or observed dropouts (e.g. 0.1%–10% sampling efficiency) and accurate imputation is needed before clustering, visualization, or pseudotime analysis.

## When NOT to use

- Small genomic datasets: SERM learns the distribution from a small fraction of the data, so the estimated distribution may diverge from the actual distribution and produce inaccurate imputation values
- Reference/observed pairs where dropouts were experimentally observed rather than simulated — the benchmark design (reference filtered to high-expression cells/genes, dropouts simulated) may not match such data
- Batch-level analyses requiring explicit batch-effect modeling — batch effects are not considered explicitly in the SERM formulation (only z-score integration post hoc)

## Inputs

- Observed dropout-affected scRNA-seq expression matrix (.mat file such as data_observed_10perc.mat), cells x genes
- Optional reference (dropout-free) expression matrix for evaluation, filtered to cells and genes with high expression values
- ROI size (e.g. [6100, 1250] for a 12,162 x 2,422 matrix) and percent overlap between successive ROIs (0.01–0.99, default 0.5)

## Outputs

- Imputed gene expression matrix Ŝ with the same q x m dimensions as the input
- Per-gene Pearson correlation coefficients between imputed and reference data
- Downstream outputs: t-SNE/UMAP/PHATE embeddings, k-means clustering indices (accuracy, NMI, AR, Hubert), TSCAN/Slingshot pseudotime

## How to apply

Load the observed dropout-affected matrix (e.g. 'data_observed_10perc.mat' via scipy.io.loadmat; rows are cells, columns are genes) and run SERM, which first denoises via an autoencoder, fits a parametric PDF (Gaussian, Rayleigh, or exponential with decay parameter λ chosen by minimum RMSE), then divides the matrix into N ≥ 4 rectangular ROIs — typically ROI height = cell count / 4 and width = gene count / 4 (e.g. [6100, 1250] for the 12,162-cell × 2,422-gene bone-marrow stroma reference) — with 0.25–0.75 percent overlap (default 0.5). Each ROI is equalized with 100 histogram bins via the inverse-transform transform T_Si(k) = −⌊(1/λ) ln(1 − F_Si(k))⌋, the ROI slides along x and y (sliding step equal to window size), and all ROIs are blended by bilinear resampling for global consistency. Compare against competing methods (MAGIC, mcImpute, DeepImpute, SAUCIE, scVI, etc.) on the same observed matrices, then validate by Pearson correlation of imputed vs. reference expression per gene and by downstream cluster-quality indices (accuracy, NMI, AR, Hubert) or pseudotime agreement. The imputed matrix Ŝ must retain the same q × m dimensions as the input.

## Related tools

- **self-consistent expression recovery machine (SERM)** (Primary imputation tool: autoencoder distribution learning + adaptive histogram equalization over sliding ROIs to recover dropout values) — https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
- **MAGIC** (Competing Markov affinity-based graph imputation method run with default configurations on the same observed matrices for benchmarking) — https://github.com/pkathail/magic
- **mcImpute** (Competing matrix-completion-based imputation method for benchmarking against SERM) — https://github.com/aanchalMongia/McImpute_scRNAseq
- **DeepImpute** (Competing deep learning imputation method for benchmarking against SERM) — https://github.com/lanagarmire/deepimpute
- **SAUCIE** (Competing sparse-autoencoder imputation method for benchmarking against SERM) — https://github.com/KrishnaswamyLab/SAUCIE
- **PHATE** (Visualization of reference, observed, and imputed data (2 dims, 5 nearest neighbors, alpha = 40, metric MDS, Euclidean, gamma = 0.5; 100 PCs zebrafish / 50 PCs EB))

## Examples

```
import scipy.io as sio; import serm as srm; dataObs = sio.loadmat('data_observed_10perc.mat')['data_observed']; out_SERM = srm.serm(dataObs, [6100,1250], 0.5, randomize=False)
```

## Evaluation signals

- Imputed matrix Ŝ has identical q x m dimensions to the observed input matrix
- Equalized ROI histograms follow the CDF of the minimum-RMSE fitted distribution (Gaussian, Rayleigh, or exponential)
- Per-gene Pearson coefficient vs. reference exceeds 0.9 for most genes at 5% sampling efficiency (competitors ≤ 0.4); 0.15–0.65 at 0.1% downsampling
- Number of zeros in the SERM-imputed data is similar to the reference data, not inflated or flattened
- Downstream cluster quality indices (accuracy, NMI, AR, Hubert) exceed those of MAGIC, mcImpute, DeepImpute, SAUCIE, and other baselines; pseudotime (TSCAN/Slingshot + monocle Pearson vs. stage labels) is highest for SERM with no manifold distortion visible in PHATE/t-SNE/UMAP

## Limitations

- SERM learns the gene expression model from a small fraction of the data — for small genomic datasets the estimated distribution may diverge from the actual distribution, leading to inaccurate imputation values
- An implicit assumption is that ROI coverage is sufficiently large that the captured ROI distribution is representative of the entire matrix
- Batch effects are not considered explicitly in the SERM formulation; multi-batch data must be integrated afterwards via z-score operation
- The benchmark reference was created by keeping only high-expression cells and genes, and dropouts were simulated (e.g. gamma/Poisson sampling per Huang et al.) rather than experimentally observed

## Evidence

- [abstract] we introduce a data-driven gene expression recovery framework, referred to as self-consistent expression recovery machine (SERM), to impute the missing expressions: "we introduce a data-driven gene expression recovery framework, referred to as self-consistent expression recovery machine (SERM), to impute the missing expressions"
- [intro] imputation is performed based on histogram equalization using the learned probability distribution: "imputation is performed based on histogram equalization using the learned probability distribution (Fig. 1-step 5)"
- [methods] The ROI is slided throughout the whole expression matrix and gene imputation is performed at each ROI: "The ROI is slided throughout the whole expression matrix and gene imputation is performed at each ROI"
- [methods] After applying the histogram equalization, we fit different probability distribution functions (PDF) to pSo: [evidence span withheld — anchor status: not_found]
- [results] SERM provides a Pearson coefficient better than 0.9 for most of the genes, which is much higher than the results (≤0.4) from most other methods: "For the case of 5% sampling efficiency, SERM provides a Pearson coefficient better than 0.9 for most of the genes, which is much higher than the results (≤0.4) from most other methods"
- [discussion] This poses a challenge in dealing with small genomic datasets, as the estimated distribution in SERM may diverge from the: "This poses a challenge in dealing with small genomic datasets, as the estimated distribution in SERM may diverge from the"
- [readme] The observed data were generated from the reference data by simulating efficiency loss that introduces zeros following the work of Huang et al.: "The observed data were generated from the reference data by simulating efficiency loss that introduces zeros
following the work of Huang et al."
