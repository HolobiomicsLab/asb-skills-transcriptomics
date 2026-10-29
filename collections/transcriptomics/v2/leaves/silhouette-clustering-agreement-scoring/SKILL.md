---
name: silhouette-clustering-agreement-scoring
description: Use when simulated scRNA-seq data (e.g. Splatter two- or six-group, 200
  genes, 2000 cells, with dropout) has ground-truth celltype labels and you must numerically
  judge whether a denoiser (DCA ZINB vs MSE autoencoder) recovers celltype structure.
when_to_use_negative:
- Real experimental scRNA-seq data with no simulated ground-truth celltype labels
  — the article notes the definition of a ground truth can be challenging for real
  data, so ARI against known labels cannot be computed
- Evaluating imputation accuracy at zero-entry level rather than celltype structure
  recovery — silhouette/ARI do not measure per-cell-gene overimputation
- Datasets where cells and genes with zero expression were not removed and library-size
  normalization has not been applied consistently across compared matrices
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3366
tools:
- name: DCA
  role: Source of the denoised matrix under evaluation (ZINB-loss deep count autoencoder
    whose celltype recovery is scored)
  repo: https://github.com/theislab/dca
- name: Scanpy
  role: PCA and clustering workflow on denoised and ground-truth matrices used for
    silhouette/ARI computation; also provides normalize_total for library-size normalization
    of DCA output
- name: anndata
  role: Loads simulated count matrices and ground-truth celltype labels into h5ad
    objects for scoring
  repo: https://github.com/theislab/anndata
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-018-07931-2
    title: Single-cell RNA-seq denoising using a deep count autoencoder
  source_content_hash: sha256:d35ed0b41a64c2c6af116cfc5c9fe1cf238d60b5685f20d10d23b8418e28da54
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/dca
  tools:
  - DCA
  - Scanpy
  - anndata
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# silhouette-clustering-agreement-scoring

## Summary

Quantifies how well a denoised scRNA-seq matrix (e.g. DCA vs MSE-autoencoder output vs no-dropout ground truth) recovers simulated celltype structure using silhouette scores and clustering-agreement metrics such as ARI against ground-truth labels.

## When to use

Use when simulated scRNA-seq data (e.g. Splatter two- or six-group, 200 genes, 2000 cells, with dropout) has ground-truth celltype labels and you must numerically judge whether a denoiser (DCA ZINB vs MSE autoencoder) recovers celltype structure.

## When NOT to use

- Real experimental scRNA-seq data with no simulated ground-truth celltype labels — the article notes the definition of a ground truth can be challenging for real data, so ARI against known labels cannot be computed
- Evaluating imputation accuracy at zero-entry level rather than celltype structure recovery — silhouette/ARI do not measure per-cell-gene overimputation
- Datasets where cells and genes with zero expression were not removed and library-size normalization has not been applied consistently across compared matrices

## Inputs

- Ground-truth simulated count matrix (Splatter, no dropout) with simulated celltype labels
- Dropout-noisy count matrices (two-group and six-group Splatter simulations, 200 genes x 2000 cells)
- Denoised matrices (DCA mean.tsv output and MSE-autoencoder reconstructions)
- Cluster assignments per cell for each condition

## Outputs

- Comparison table of silhouette and clustering-agreement values per condition (e.g. simulation_denoising_metrics.csv)
- PCA plots colored by simulated celltype (e.g. simulation_pca.png)
- Silhouette scores and ARI values for ground truth, DCA-denoised, and MSE-denoised matrices

## How to apply

Run PCA on the ground-truth (no-dropout), DCA-denoised (ZINB loss, three hidden layers of 64/32/64 neurons), and MSE-autoencoder-denoised matrices and color cells by simulated celltype. Compute silhouette scores on each denoised/ground-truth matrix and clustering agreement (e.g. adjusted Rand index) between cluster assignments and ground-truth labels for every condition. Write the comparison to a metrics table (e.g. simulation_denoising_metrics.csv) covering both group counts, both denoisers, and the ground truth. Accept the evaluation if DCA's celltype recovery metrics are at least as high as the MSE autoencoder's, demonstrating that the ZINB count-aware loss better recovers the no-dropout structure.

## Related tools

- **DCA** (Source of the denoised matrix under evaluation (ZINB-loss deep count autoencoder whose celltype recovery is scored)) — https://github.com/theislab/dca
- **Scanpy** (PCA and clustering workflow on denoised and ground-truth matrices used for silhouette/ARI computation; also provides normalize_total for library-size normalization of DCA output)
- **anndata** (Loads simulated count matrices and ground-truth celltype labels into h5ad objects for scoring) — https://github.com/theislab/anndata

## Examples

```
from sklearn.metrics import silhouette_score, adjusted_rand_score; sil = silhouette_score(X_dca, labels); ari = adjusted_rand_score(clusters_dca, truth_labels)
```

## Evaluation signals

- The metrics table contains silhouette and clustering-agreement values for both group counts (two and six celltypes), both denoisers (DCA and MSE autoencoder), and the ground truth
- DCA's celltype recovery metrics (silhouette, ARI) are at least as high as the MSE autoencoder's on the dropout simulations
- Ground-truth (no-dropout) silhouette/ARI values represent the upper bound; denoised values should approach them more than the raw dropout matrix does
- Denoised matrices have the same dimensions as the input (excluding zero-expression genes/cells) so silhouette scores are computed on comparable cell sets

## Limitations

- It may be difficult to determine in general when denoising improves scRNA-seq data; silhouette/ARI on simulations is one proxy, not a universal criterion
- Evaluation of denoising is difficult because defining a ground truth for real data is challenging — this scoring is only straightforward on simulated data with known celltype labels
- Overimputation from inadequate hyperparameter choices (e.g. too low-dimensional bottleneck layer) can distort celltype structure in ways structure-level metrics may not fully capture

## Evidence

- [discussion] Note that in general, it may be difficult to determine when denoising improves scRNA-seq data.: "Note that in general, it may be difficult to determine when denoising improves scRNA-seq data."
- [discussion] Ground truth challenge for real data: "The evaluation of denoising is difficult because the definition of a ground truth can be challenging for real data."
- [discussion] Denoised output definition: "The denoised matrix is generated by replacing the original count values with the mean of the negative binomial component"
- [intro] DCA default architecture: "The deep learning framework (by default three hidden layers with 64, 32, 64 neurons) of DCA enables the capturing of the complexity and non-linearity in scRNA-seq data."
- [readme] mean.tsv output format: "`mean.tsv` is the main output of the method which represents the mean parameter of the ZINB distribution. This file has the same dimensions as the input file"
