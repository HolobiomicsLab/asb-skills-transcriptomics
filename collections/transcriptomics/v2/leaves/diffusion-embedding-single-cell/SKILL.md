---
name: diffusion-embedding-single-cell
description: Use when comparing imputed vs. observed scRNA-seq data for trajectory
  preservation — e.g., after SERM or MAGIC imputation of zebrafish development or
  EB differentiation matrices — to check whether the developmental manifold and state
  transitions survive imputation.
when_to_use_negative:
- When the goal is clustering accuracy on simulated multi-class data where t-SNE/UMAP
  with k-means (n=1000 initializations) is the prescribed evaluation, not PHATE manifold
  comparison
- When the input is already a low-dimensional embedding — PHATE must be run on PCs
  derived from the expression matrix
- On very small genomic datasets, where SERM's learned distribution may diverge from
  the actual data distribution and the resulting imputation (and thus the embedding
  comparison) is unreliable
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: PHATE
  role: Diffusion-based embedding used to visualize reference, observed, and imputed
    data (2 dims, 5 NN, alpha=40, metric MDS, Euclidean, gamma=0.5) and detect manifold
    distortion
- name: MAGIC
  role: Competing diffusion-based imputation method whose output is embedded to demonstrate
    visible manifold distortion relative to SERM
  repo: https://github.com/pkathail/magic
- name: self-consistent expression recovery machine (SERM)
  role: Imputation method producing the matrices whose PHATE embedding should reproduce
    the reference developmental trajectory
  repo: https://github.com/xinglab-ai/self-consistent-expression-recovery-machine
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-34595-w
    title: Leveraging data-driven self-consistency for high-fidelity gene expression
      recovery
  source_content_hash: sha256:0d5b2718ac6aa3e259f2a472258c527c42d3b9bb8d37150f7fd27325168f7668
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pkathail/magic
  tools:
  - MAGIC
  - PHATE
  - Slingshot
  - TSCAN
  - self-consistent expression recovery machine (SERM)
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# diffusion-embedding-single-cell

## Summary

Uses diffusion-based manifold embedding (PHATE) to visualize reference, dropout-affected, and imputed single-cell RNA-seq matrices, verifying that imputation (e.g., SERM) preserves the continuous developmental manifold where competing methods visibly distort it.

## When to use

Use when comparing imputed vs. observed scRNA-seq data for trajectory preservation — e.g., after SERM or MAGIC imputation of zebrafish development or EB differentiation matrices — to check whether the developmental manifold and state transitions survive imputation.

## When NOT to use

- When the goal is clustering accuracy on simulated multi-class data where t-SNE/UMAP with k-means (n=1000 initializations) is the prescribed evaluation, not PHATE manifold comparison
- When the input is already a low-dimensional embedding — PHATE must be run on PCs derived from the expression matrix
- On very small genomic datasets, where SERM's learned distribution may diverge from the actual data distribution and the resulting imputation (and thus the embedding comparison) is unreliable

## Inputs

- Reference (dropout-free) scRNA-seq expression matrix from the paper's Google Drive deposit
- Observed dropout-affected scRNA-seq matrix (e.g., zebrafish development, EB differentiation)
- Imputed expression matrices from SERM and competing methods (MAGIC, mcImpute, etc.)
- PCA-reduced matrices: top 100 PCs (zebrafish) or top 50 PCs (EB)

## Outputs

- 2D PHATE embedding coordinates for each matrix (reference, observed, imputed)
- Qualitative manifold-preservation comparison figures (reference vs. imputed trajectories)
- Supporting pseudotime orderings from TSCAN/Slingshot for the same matrices

## How to apply

Reduce each matrix (reference, observed, SERM-imputed, MAGIC-imputed) to its leading principal components first — 100 PCs for the zebrafish data (38,731 cells, 12 stages) and 50 PCs for the EB differentiation data. Embed these PCs with PHATE using 2 embedding dimensions, 5 nearest neighbors, alpha = 40, metric MDS, Euclidean distance, and gamma = 0.5. Visually compare whether the imputed embedding reproduces the continuous developmental trajectory of the reference (Fig. 5 pattern), while checking that competing imputations such as MAGIC distort the manifold. Pair the embedding with TSCAN/Slingshot pseudotime and compare pseudotime-vs-stage Pearson correlation to confirm the qualitative reading. Rationale: diffusion-based embedding is sensitive to manifold distortion, making it a validation surface for imputation fidelity.

## Related tools

- **PHATE** (Diffusion-based embedding used to visualize reference, observed, and imputed data (2 dims, 5 NN, alpha=40, metric MDS, Euclidean, gamma=0.5) and detect manifold distortion)
- **MAGIC** (Competing diffusion-based imputation method whose output is embedded to demonstrate visible manifold distortion relative to SERM) — https://github.com/pkathail/magic
- **self-consistent expression recovery machine (SERM)** (Imputation method producing the matrices whose PHATE embedding should reproduce the reference developmental trajectory) — https://github.com/xinglab-ai/self-consistent-expression-recovery-machine

## Examples

```
import phate; G = phate.PHATE(n_components=2, knn=5, alpha=40, mds='metric', distance='euclidean', gamma=0.5); emb = G.fit_transform(X_pca)
```

## Evaluation signals

- SERM-imputed PHATE embedding reproduces the continuous developmental trajectory of Fig. 5 without distortion, while MAGIC-imputed data visibly distorts the manifold
- Monocle pseudotime on SERM-imputed data achieves consistently higher Pearson correlation with ground-truth zebrafish stage labels than all competing imputation methods across sampling efficiencies
- Correct PCA dimensionality used per dataset: 100 PCs for zebrafish, 50 PCs for EB
- PHATE parameters match the paper: 2 embedding dimensions, 5 nearest neighbors, alpha = 40, metric MDS, Euclidean distance, gamma = 0.5
- Reference-to-imputed Pearson coefficients are highest for SERM on both zebrafish and EB datasets

## Limitations

- Dropouts in the evaluation data were simulated (following Huang et al. sampling at 1%–10% efficiency) rather than experimentally observed, and reference data were filtered to cells and genes with high expression
- Diffusion-based imputation methods like MAGIC can degrade data as a result of imputation, so embedding differences may reflect imputation artifacts rather than biology
- SERM's assumption that ROI coverage is sufficiently large for the captured distribution to represent the entire matrix may fail on small genomic datasets

## Evidence

- [methods] PHATE parameters and PC counts: [evidence span withheld — anchor status: not_found]
- [results] MAGIC distortion of the manifold: "t-SNE and UMAP results from SERM imputed data are much better in separating the classes, whereas MAGIC degrades the data as a result of imputation."
- [readme] MAGIC as denoising algorithm: [evidence span withheld — anchor status: not_found]
- [abstract] SERM scale and speed advantage: "We show that SERM improves the accuracy of gene imputation with orders of magnitude enhancement in computational efficiency"
