---
name: dropout-perturbation-benchmarking
description: Use when you must test whether a denoising method preserves or recovers
  biological structure (celltypes) in scRNA-seq data corrupted by dropout, using simulated
  ground truth rather than real data.
when_to_use_negative:
- Real scRNA-seq data with no reliable ground truth — the article notes evaluation
  of denoising is difficult because defining a ground truth for real data is challenging
- Questions about differential expression or trajectory accuracy rather than celltype
  structure recovery — this benchmark only assesses celltype separation metrics
- Datasets without dropout perturbation (no-dropout ground-truth simulations) — the
  contrast being tested requires the noisy vs. clean pair
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: DCA
  role: Count-aware denoiser using the ZINB loss; generates the denoised matrix by
    replacing original counts with the predicted negative binomial mean
  repo: https://github.com/theislab/dca
- name: Scanpy
  role: Preprocessing/filtering and PCA/visualization of ground-truth and denoised
    matrices in the benchmark comparison
  repo: https://scanpy.readthedocs.io/en/stable/api/scanpy.pp.normalize_total.html
- name: anndata
  role: H5ad-formatted container for simulated count matrices and celltype labels
    loaded into the workflow
  repo: https://github.com/theislab/anndata
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-018-07931-2
    title: Single-cell RNA-seq denoising using a deep count autoencoder
  source_content_hash: sha256:9f679648dcc2ffa1d2400e8a7f7a7d8c7128ecd52b5640ce4bb3adebeed7ab8c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/dca
  tools:
  - DCA
  - Scanpy
  - anndata
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# dropout-perturbation-benchmarking

## Summary

Benchmark scRNA-seq denoising methods under controlled dropout perturbation by simulating count data (e.g., with Splatter), applying methods such as DCA with a ZINB count loss versus a plain MSE autoencoder, and comparing celltype recovery against the no-dropout ground truth.

## When to use

Use when you must test whether a denoising method preserves or recovers biological structure (celltypes) in scRNA-seq data corrupted by dropout, using simulated ground truth rather than real data.

## When NOT to use

- Real scRNA-seq data with no reliable ground truth — the article notes evaluation of denoising is difficult because defining a ground truth for real data is challenging
- Questions about differential expression or trajectory accuracy rather than celltype structure recovery — this benchmark only assesses celltype separation metrics
- Datasets without dropout perturbation (no-dropout ground-truth simulations) — the contrast being tested requires the noisy vs. clean pair

## Inputs

- Splatter-simulated scRNA-seq count matrices (e.g., two or six celltypes, 200 genes, 2000 cells)
- Dropout-perturbed count matrices generated with the Methods dropout model
- Ground-truth celltype labels for the simulated cells
- AnnData h5ad objects holding counts and labels

## Outputs

- Denoised count matrices (e.g., DCA mean.tsv / mean_norm.tsv) for each denoiser and condition
- PCA plots colored by simulated celltype (simulation_pca.png)
- Comparison table of silhouette scores and clustering agreement (ARI) per condition and denoiser (simulation_denoising_metrics.csv)

## How to apply

Simulate two-group and six-group scRNA-seq count data with Splatter (200 genes, 2000 cells) and apply a dropout model to produce noisy count matrices alongside the no-dropout ground truth. Load the simulated matrices and ground-truth celltype labels into AnnData h5ad objects, then denoise the dropout data with DCA using the zero-inflated negative binomial (ZINB) loss and the default network (three hidden layers with 64, 32, 64 neurons, RMSProp, learning rate 0.001, learning rate reduced by 0.1 after 20 epochs without improvement, early stopping after 25 epochs). Train a plain MSE-reconstruction autoencoder on the same dropout matrices as the baseline. Run PCA on ground-truth, DCA-denoised, and MSE-denoised matrices with cells colored by simulated celltype, then compute silhouette scores and clustering agreement (e.g., ARI) against ground-truth labels. Accept the benchmark if DCA's celltype-recovery metrics are at least as high as the MSE autoencoder's, demonstrating that the count-aware ZINB loss better recovers the no-dropout structure.

## Related tools

- **DCA** (Count-aware denoiser using the ZINB loss; generates the denoised matrix by replacing original counts with the predicted negative binomial mean) — https://github.com/theislab/dca
- **Scanpy** (Preprocessing/filtering and PCA/visualization of ground-truth and denoised matrices in the benchmark comparison) — https://scanpy.readthedocs.io/en/stable/api/scanpy.pp.normalize_total.html
- **anndata** (H5ad-formatted container for simulated count matrices and celltype labels loaded into the workflow) — https://github.com/theislab/anndata

## Examples

```
dca dropout_sim.h5ad results_dca; python -c "import scanpy as sc; a=sc.read_h5ad('results_dca/denoised.h5ad'); sc.pp.pca(a); sc.tl.louvain(a); print(sklearn.metrics.adjusted_rand_score(a.obs['celltype'], a.obs['louvain']))"
```

## Evaluation signals

- Comparison table contains silhouette and clustering-agreement (ARI) values for both group counts (two and six celltypes), both denoisers, and the ground truth
- DCA's celltype-recovery metrics (silhouette, ARI) are at least as high as the MSE autoencoder's on dropout data
- Denoised output dimensions match the input after excluding zero-expression genes/cells, as in DCA's mean.tsv
- PCA of the DCA-denoised matrix visually recovers the simulated celltype clusters seen in the no-dropout ground truth
- DCA training converges under the stated regime: RMSProp, lr 0.001, reduced by 0.1 after 20 epochs without validation-loss improvement, early stopping after 25 epochs

## Limitations

- Evaluation of denoising is difficult because defining a ground truth for real data is challenging; conclusions hold for the simulated dropout setting
- Overimputation may occur with inadequate hyperparameter choices such as too low-dimensional a bottleneck layer
- Hyperparameters were not fine-tuned for any of the analyses described in the manuscript, so results reflect default settings

## Evidence

- [results] Count-based loss necessity finding: [evidence span withheld — anchor status: not_found]
- [abstract] DCA count-distribution noise model: "DCA takes the count distribution, overdispersion and sparsity of the data into account using a negative binomial noise model with or without zero-inflation"
- [discussion] Training regime: "We used RMSProp for optimization with learning rate 0.001. Learning rate is multiplied by 0.1 if validation loss does not improve for 20 epochs."
- [intro] Default architecture: "The deep learning framework (by default three hidden layers with 64, 32, 64 neurons) of DCA enables the capturing of the complexity and non-linearity in scRNA-seq data."
- [readme] DCA main output file: "`mean.tsv` is the main output of the method which represents the mean parameter of the ZINB distribution."
