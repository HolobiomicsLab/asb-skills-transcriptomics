---
name: gaussian-mixture-model-subclone-separability
description: Use when comparing how accurately different cell-cycle correction methods
  (Cyclum, Seurat, ccRemover, PC1–5) preserve or recover subclone structure in scRNA-seq
  data containing two known subpopulations.
when_to_use_negative:
- Data contain more than two subclones or the subclone identity is unknown — the two-component
  GMM assumes exactly two delineable populations
- Cell-cycle is not a confounder in the data (e.g., quiescent/post-mitotic cells),
  so correction and separability benchmarking is unnecessary
- Only raw, uncorrected expression is of interest — separability on uncorrected data
  serves only as a baseline, not the target output
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: Cyclum
  role: Nonlinear cell-cycle correction based on circular manifold projection with
    a sinusoidal autoencoder; the corrected data are fed to the GMM separability evaluation
  repo: https://github.com/KChen-lab/Cyclum
- name: Seurat
  role: Linear-regression cell-cycle correction baseline whose outputs are evaluated
    with the same two-component GMM separability metric
- name: ccRemover
  role: Linear-regression cell-cycle correction baseline evaluated with the same GMM
    separability metric
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-15295-9
    title: Latent periodic process inference from single-cell RNA-seq data
  source_content_hash: sha256:d319e85fb33a2157945d626989fd1977700505b43c3332511c670c7fa8e7571f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KChen-lab/Cyclum
  tools:
  - Cyclum
  - Seurat
  - ccRemover
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# Gaussian mixture model subclone separability

## Summary

Quantifies how well two cell subclones can be delineated in cell-cycle-corrected scRNA-seq data by fitting a two-component Gaussian mixture model (GMM) on leading components and scoring separability, as used to benchmark Cyclum against Seurat, ccRemover, and PC1–5 correction on virtual tumor samples.

## When to use

Use when comparing how accurately different cell-cycle correction methods (Cyclum, Seurat, ccRemover, PC1–5) preserve or recover subclone structure in scRNA-seq data containing two known subpopulations.

## When NOT to use

- Data contain more than two subclones or the subclone identity is unknown — the two-component GMM assumes exactly two delineable populations
- Cell-cycle is not a confounder in the data (e.g., quiescent/post-mitotic cells), so correction and separability benchmarking is unnecessary
- Only raw, uncorrected expression is of interest — separability on uncorrected data serves only as a baseline, not the target output

## Inputs

- mESC scRNA-seq expression matrix (preprocessed as scaled (variance=1), centered (mean=1) log2(x+1) data)
- Virtual tumor samples constructed by ×2 perturbation of gene sets (including the >400-gene cell-cycle set)
- Cell-cycle-corrected expression matrices from Cyclum, Seurat, ccRemover, and PC1–5 correction

## Outputs

- Two-component GMM subclone assignments per cell for each corrected dataset
- Subclone separability metrics per correction method
- Aggregated comparison table subclone_separability.csv

## How to apply

Preprocess the mESC scRNA-seq matrix with a log2(x+1) transformation, centering to mean = 1 and scaling to variance = 1. Generate virtual tumor samples by doubling (×2) the expression of perturbed gene sets — including the >400-gene cell-cycle set — across mESC cells, then correct cell-cycle effects with each method being compared (Cyclum via circular-manifold projection, Seurat/ccRemover via linear regression, PC1–5 as a linear baseline). For each corrected dataset, fit a two-component Gaussian mixture model on the leading components and quantify the separability of the two subclones. Aggregate the GMM separability metrics across all methods into a comparison table (subclone_separability.csv). The rationale is that the circular (nonlinear) approach should better deconvolve the cell-cycle confounder and thus yield higher subclone separability than linear baselines, particularly when more than 400 cell-cycle genes are perturbed.

## Related tools

- **Cyclum** (Nonlinear cell-cycle correction based on circular manifold projection with a sinusoidal autoencoder; the corrected data are fed to the GMM separability evaluation) — https://github.com/KChen-lab/Cyclum
- **Seurat** (Linear-regression cell-cycle correction baseline whose outputs are evaluated with the same two-component GMM separability metric)
- **ccRemover** (Linear-regression cell-cycle correction baseline evaluated with the same GMM separability metric)

## Examples

```
from cyclum.postproc import refine_labels; refine_labels(pseudotime=[4, 5, 1, 2, 3, 6, 7, 8], original_labels=['G1', 'G1', 'G1', 'G1', 'S', 'S', 'G2M', 'G2M'])
```

## Evaluation signals

- The two-component GMM separability ranking across methods reproduces the published Fig. 3c / Source Data pattern, with Cyclum (circular approach) above Seurat, ccRemover, and PC1–5
- Cyclum achieves significantly higher separability than linear baselines, especially when >400 cell-cycle genes are perturbed in the virtual tumor data
- Uncorrected data serve as the reference baseline; each corrected dataset should show altered separability relative to this uncorrected baseline
- subclone_separability.csv is produced with one separability metric per method across all perturbation conditions
- Preprocessed input satisfies the normalization invariants: log2(x+1), mean = 1, variance = 1

## Limitations

- Cyclum's advantage over linear methods is largest when more than 400 cell-cycle genes are perturbed; with few perturbed cell-cycle genes the gain over linear regression baselines may be marginal
- Linear representations (Seurat, ccRemover, PC1–5) cannot accurately represent the circular timing of periodic processes, so conclusions from this benchmark concern periodic confounders
- Cyclum requires fitting data to predefined circular manifolds and, in practice, Python/Keras/TensorFlow with specific tested versions (e.g., tensorflow 1.14.0); newer TensorFlow 2.x is not listed as tested

## Evidence

- [other] Under almost all conditions, Cyclum achieved significantly higher subclone-separability accuracy than Seurat, ccRemover, and PC1~5: [evidence span withheld — anchor status: not_found]
- [methods] scaled, centered, log-transformed preprocessing: "scaled (variance = 1), centered (mean = 1) log-transformed (:math:`\log_2(x+1)`) data work well."
- [methods] Cyclum projects data to a circular manifold rather than using marker genes: "The methodology projects high dimensional gene expression data to a circular manifold, instead of using marker genes."
- [abstract] Applying Cyclum to removing cell-cycle effects substantially improves delineations of cell subpopulations: "Applying Cyclum to removing cell-cycle effects substantially improves delineations of cell subpopulations"
- [readme] Cyclum provides methods to remove cell cycle effects: [evidence span withheld — anchor status: external_doc]
